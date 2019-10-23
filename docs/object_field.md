---
title: 字段
---

Steedos 默认使用MongoDB数据库，相比传统的SQL数据库，可以额外实现数组、内嵌对象、内嵌表格等高级字段类型。

## 字段名 name
字段在数据库中保存的名称，区分大小写，建议全部使用小写字母。

当使用OData或GraphQL API查询和更新对象数据时，也使用字段名。

例如以下例子定义了一个owner字段:
```yaml
owner:
  label: 责任人
  type: lookup
  reference_to: users
  defaultValue:
  required: true
  inlineHelpText: 请选择此任务的责任人，相关人员会收到待办提醒。
  is_wide: false
  searchable: true
  index: true
  omit: false
  hidden: false
```

## 字段属性
字段名下一层，可以配置属性，定义字段的功能和界面操作。

### 显示名 label
字段在最终用户界面上的显示名称。显示名称支持国际化，如果系统检测到i18n文件中包含 "{objectname}_{fieldname}"，以翻译为准。

### 默认值 defaultValue
设定字段的默认值，可以是固定值，也可以可配置默认值[公式](object_field_formula.md)，例如 {userId}, {spaceId} 等
```yaml
owner:
  defaultValue: {userId}
```

### 必填 required
设定当前字段必填，在新增和修改界面如果当前字段未填写则不能报错。当调用OData接口操作数据时，必填字段也必须传入。

### 帮助文本 inlineHelpText
可配置表单填写时显示的一段描述，帮助业务人员理解该字段的作用。

在表单填写时，在字段名右侧显示为一个Info图标，业务人员点击此图标可以看到帮助文本。

### 宽字段 is_wide
对于宽屏幕，Steedo显示表单时会显示为两列。宽字段始终占两列宽度，窄字段占一列宽度。

### 分组 group
在记录的显示页面和编辑页面，将字段分组显示。

group值相同的字段被分到同一组，分组标题为group的值，分组内的字段顺序为字段在表单上定义时的先后顺序。

### 多选 multiple
文本、选择类、lookup、master/detail 型字段添加此属性，可以实现多选功能，数据库中保存也是对应的数组类型。

### 可搜索 seachable
系统进行快捷搜索时，默认只搜索name字段的内容。如果配置了此属性，当用户在此对象中执行搜索时，会同时搜索此字段的内容。

注意，对lookup类型的字段如果配置了 searchable，不能同时搜索相关表中的name属性。

### 可排序 sortable
考虑到数据库的性能，只有设定了可排序的字段，在列表视图中才能按照此字段进行排序。
可排序字段系统会自动创建索引。默认为不可排序

### 索引 index
配置是否在数据库中为此字段创建索引，默认为不创建索引。对于大多数数据库引擎，索引字段配置的太多会导致数据库性能下降。

通常建议为以下类型的字段配置索引：
- 需要搜索的字段
- 在列表视图或报表中需要过滤的字段
- 相关表的字段

### 唯一 unique
为此字段创建唯一索引，字段值在数据库中不得重复。

### 只读 readonly
此字段只显示在查看页面或列表页面上，新增和修改页面都不显示。（此属性即将作废，不建议使用。）

### 隐藏 hidden
对于一些后台计算用的字段，可以设置为隐藏。隐藏字段在列表、查看、编辑、过滤界面等都不显示，但是可以通过脚本操作，也可以配置在过滤条件中。

### 编辑时忽略 omit
只是新建和编辑表单中不显示，列表、表单详细界面等可能显示。

### 标题字段 is_name
系统默认字段名为name的字段为标题字段，在列表显示时，标题字段会自动加上链接，点击进入记录查看界面。

如果当前表没有name字段，需要指定其他字段为标题字段，可以设置此属性。

### 禁用 disabled
在编辑时禁用此字段。（此属性即将作废，不建议使用。）

### 黑箱字段 blackbox
如果配置了此属性，Steedos在验证数据格式时，忽略此字段的内容。

### 值范围 allowedValues
使用数组格式定义此字段的的可选项范围。如果配置了此属性，不仅在表单上填写数据会校验，通过API接口操作数据，或是编写触发器操作数据时，也会做校验。
```yaml
priority:
  allowedValues:
    - high
    - normal
    - low
```

![表单编辑效果](assets/field_guide.png)

## 第三方数据源

### 主键 primary
默认数据源使用mongodb数据库，默认使用_id作为主键。如使用第三方SQL数据源，需要手工指定主键字段。

## 系统字段
如果使用Steedos默认数据源，每个Steedos对象都会自动创建一些系统字段。

### 所属工作区 space
Steedos可以分工作区保存用户数据，每个用户可以属于多个工作区。
space字段值在记录创建时生成，默认为当前选中的工作区。
当在视图中定义了 filter_scope: space 时，自动按照此字段过滤。

### 所属责任人 owner
owner字段用于保存当前记录的责任人，例如合同的经办人，客户的负责人等。
当在视图中定义了 filter_scope: mine 时，自动按照此字段过滤。
字段值记录创建时自动生成，默认为当前用户。如果需要让用户自主选择，可以在代码中增加配置。
```yaml
  owner: 
    label: 责任人
    omit: false
    hidden: false
```

### 所属单位 company_id
Steedos可以控制单位级别权限，授权默写用户只能查看/修改本单位的数据。
company_id字段用于标记记录责任人的所属单位，默认为隐藏(hidden)自动赋值。
如果需要由用户在界面上选择，可以增加如下配置覆盖此字段属性。
```yaml
  company_id: 
    label: 所属单位
    omit: false
    hidden: false
```
### 审计相关
以下字段用于记录审计相关信息，系统会自动赋值。
- 创建日期(created)：记录创建时生成，默认为当前时间
- 创建人(created_by)：记录创建时生成，默认为当前用户
- 修改日期(modified)：记录修改时生成，默认为当前时间
- 修改人(modified_by)：记录修改时生成，默认为当前用户
- 已删除(is_deleted)：字段类型为布尔(boolean)，默认为隐藏(hidden)、编辑时忽略(omit)
- 删除日期(deleted)：默认为隐藏(hidden)、编辑时忽略(omit)
- 删除人(deleted_by)：默认为隐藏(hidden)、编辑时忽略(omit)

### 审批相关
- 已锁定(locked)：字段类型为布尔(boolean)，默认为隐藏(hidden)、编辑时忽略(omit)
- 记录的相关审批单(instances)：默认为隐藏(hidden)、编辑时忽略(omit)