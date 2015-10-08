title: underscore template
date: 2015-09-30 10:52:25
tags:
categories: javascript
---

对于前端模板中的`<%= ... %>`, `<% ... %>`, `<%- ... %>` 一直傻傻分不清楚。所以特地写了一篇这个文章，介绍一下他们的用法，方便以后查阅。
主要参考了[underscore的tmeplate](http://underscorejs.org/#template).

<!--more-->

## 执行js代码 '<% ... %>'

'<% ... %>'的作用是可以执行任何的javascript代码
看下面这个例子

```
var compiled = _.template("<% print('Hello ' + epithet); %>");
compiled({epithet: "stooge"});
=> "Hello stooge"
```

可以看一个复杂的，带了控制流的，这个例子中，使用了for语句，响应的html元素是包含在两个'<% ... %>' tag中的
```
<tr class="alarm_metric_tr">
  <td>
    <select id="id_metric_name" name="metric_name" class="metric_name">
      <% for(index=0; index<metric_choices.length; index++){ %>
        <option value="<%- metric_choices[index][0] %>"><%- metric_choices[index][1] %></option>
      <% } %>
    </select>
  </td>
</tr>
```
当然也可以使用if...else...这种语句，可以参考后面复杂的例子

## 替换变量`<%= ... %>`

这个就比较简单了，直接替换响应的变量

```
var compiled = _.template("hello: <%= name %>");
compiled({name: 'moe'});
=> "hello: moe"
```

## 替换变量时escape '<%- ... %>'

与前面的tag比较，最大的变化是这个替换以后，会escape其内容，主要是从安全方面考虑，防止在html中插入代码等

```
var template = _.template("<b><%- value %></b>");
template({value:'<script>'});
=> "<b>&lt;script&gt;</b>"
```

## 复杂的例子

```
<div class="monitor_graph" id="<%- instance_id %>">
  <div class="graph_title">
    <%- instance_name %>
    <a href="javascript:;" class="btn_type btn_delete delete_monitor_graph">Delete<span class="btn_bg"></span></a>
  </div>
  <div class="resources_overview">
      <div class="overview_line_chart" id="cpu">
        <img src="/base/img/loading.gif" alt="" class="small_loading">
      </div>
      <div class="overview_line_chart" id="memory">
        <img src="/base/img/loading.gif" alt="" class="small_loading">
      </div>
      <div class="overview_line_chart" id="disk">
        <img src="/base/img/loading.gif" alt="" class="small_loading">
      </div>
      <div class="overview_line_chart" id="disk_traffic">
        <img src="/base/img/loading.gif" alt="" class="small_loading">
      </div>
      <div class="overview_line_chart" id="network_traffic">
        <img src="/base/img/loading.gif" alt="" class="small_loading">
      </div>
  </div>
</div>
```

这是一个结合了for和if等复杂控制流的模板。

```
<% for (var i in table) { %>
  <% var row = table[i] %>
  <% if (row.project_pays.length === 0) { %>
      <tr>
        <td class="txt_cent ver_top"><strong><%= row.month %></strong></td>
        <td class="bd_lft2 txt_lft pd_lft"><strong class="ft_blue">TOTAL</strong></td>
        <td class="txt_rgt pd_rgt"><strong class="ft_blue"><%= row.total %>doller</strong></td>
      </tr>
  <% } else { %>
      <tr>
        <td rowspan="<%= row.project_pays.length + 1 %>" class="txt_cent ver_top"><strong><%= row.month %></strong></td>
        <td class="bd_lft2 txt_lft pd_lft"><%= row.project_pays[0].project_name %></td>
        <td class="txt_rgt pd_rgt"><%= row.project_pays[0].pay %>doller</td>
      </tr>
      <% for (var j in row.project_pays) { %>
        <% if (j != 0) { %>
        <tr>
          <td class="bd_lft2 txt_lft pd_lft"><%= row.project_pays[j].project_name %></td>
          <td class="txt_rgt pd_rgt"><%= row.project_pays[j].pay %>doller</td>
        </tr>
        <% } %>
      <% } %>
      <tr>
        <td class="bd_lft2 txt_lft pd_lft"><strong class="ft_blue">TOTAL</strong></td>
        <td class="txt_rgt pd_rgt"><strong class="ft_blue"><%= row.total %>doller</strong></td>
      </tr>
  <% } %>
<% } %>
```