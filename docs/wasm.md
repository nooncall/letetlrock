# WASM

## proxy-wasm

### proxy-wasm-go-host

一些核心结构

- common/vm.go 对wasm的接口 (实现是Wasmer)
  - WasmVM
  - WasmModule
  - WasmInstance
  - WasmFunction
- common/buffer.go

## DM

### Syncer

核心函数

- Syncer.handleRowsEvent

核心过滤器/路由器

- bf "github.com/pingcap/tidb-tools/pkg/binlog-filter" binlog过滤
- cm "github.com/pingcap/tidb-tools/pkg/column-mapping" 列名/列值转换
- router "github.com/pingcap/tidb-tools/pkg/table-router" 表名路由

可做成WASM插件的一些点

- skipRowsEvent -> filter.IsSystemSchema, binlog_filter.Filter
- mappingDML -> HandleRowValue
- generateExtendColumn -> table_router Table.FetchExtendColumn

痛点需求:

- 用wasm实现Exprs, 对列名/列值进行修改 (主要是列值)
- 用wasm实现扩展列值填充

## 设计上的思考

### 过滤器配置与WASM配置

wasm模块有可能需要做成"半定制化"的, 即: 一些核心配置仍用配置文件定义, 只开放少量的扩展点.

那这样的话, 如何创建并生成一个wasm Module?

- 是动态生成一个module, 把配置一起打包进去
- 用模板生成module, 启动时加载配置

## 踩过的坑

### TinyGo兼容性问题

尝试直接把tidb-tools pkg下面的一些包拷贝到proxy-wasm-go-sdk下面, 期望能直接复用已有处理逻辑, 但出现了一些问题.

tinygo对反射支持不太好, 有些地方不太好处理.

1. json.Unmarshal用不了, 下面这个图的写法会报panic.
2. github.com/pkg/errors, github.com/pingcap/errors用不了, 需要换成errors
3. 接口类型断言用不了
