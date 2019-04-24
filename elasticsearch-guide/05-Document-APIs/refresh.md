## `?refresh`
[Index](../05-Document-APIs/Index-API.md)，[Update](../05-Document-APIs/Update-API.md)，[Delete](../05-Document-APIs/Delete-API.md)和[Bulk](../05-Document-APIs/Bulk-API.md) API支持设置`refresh`，以控制此请求所做的更改何时对搜索可见。以下是可选值：

**空字符串或`true`**
    在操作发生后，立即刷新相关的主分片和副本分片（而不是整个索引），以便更新的文档立即显示在搜索结果中。**只有**在从索引和搜索角度进行仔细考虑并验证它不会导致性能不佳之后，才能进行此操作。

**`wait_for`**
    在回复之前，请等待刷新请求所做的更改。这不会强制立即刷新，而是等待刷新发生。 Elasticsearch会自动刷新已更改每个`index.refresh_interval`的分片，默认为一秒。那个设置是[动态](../15-Index-Modules/README.md#动态索引设置)的。在任何支持它的API上调用[刷新](../08-Indices-APIs/Refresh.md) API或将`refresh`设置为`true`也会导致刷新，从而导致已经运行的请求返回`refresh=wait_for`。

**`false`**（默认值）
    不采取与刷新相关的操作。此请求所做的更改将在请求返回后的某个时间点显示。
    
### 选择要使用的设置
除非您有充分的理由等待更改变为可见，否则请始终使用`refresh=false`，或者，因为这是默认值，只需不在URL增加`refresh`参数。这是最简单快捷的选择。

如果您必须让请求所做的更改与请求同步显示，那么您必须选择在Elasticsearch上添加更多负载（`true`），并等待响应更长时间（`wait_for`）。以下是应该注意的几点：

- 对索引进行的更改越多，`wait_for`保存的工作量与`true`相比就越多。如果索引仅在每个`index.refresh_interval`中更改一次，那么它将不会保存任何工作。
- `true`会创建效率较低的索引构造（小段），这些构造稍后必须合并为更有效的索引构造（更大的段）。这意味着开销发生在索引时创建小段，在搜索时搜索小段，并在合并时创建较大的段。
- 永远不要连续启动多个`refresh=wait_for`请求，应使用`refresh=wait_for`将它们处理为单个批量请求，Elasticsearch将并行启动它们并仅在它们全部完成时返回。
- 如果刷新间隔设置为`-1`，禁用自动刷新，则具有`refresh=wait_for`的请求将无限期地等待，直到某个操作导致刷新。相反，将`index.refresh_interval`设置为短于200ms的默认值，将使`refresh=wait_for`更快恢复，但它仍会生成效率低下的段。
- `refresh=wait_for`仅影响它所在的​​请求，但是，通过立即强制刷新，`refresh=true`将影响其他正在进行的请求。通常，如果您有一个正在运行的系统，您不希望打扰，那么`refresh=wait_for`是一个较小的修改。

### `refresh=wait_for`可强制刷新
如果在`index.max_refresh_listeners`（默认为1000）的请求等待刷新时，出现`refresh=wait_for`请求，则该请求的行为就像刷新设置为`true`一样：它将强制刷新。这保证了当`refresh=wait_for`请求返回其更改对于搜索可见时，同时防止被阻止请求的未检查资源被使用。如果请求强制刷新，因为它用完了侦听器插槽，那么它的响应将包含`"forced_refresh":true`。

批量请求只会占用他们接触的每个分片上的一个插槽，无论他们修改分片多少次。

### 示例
这些将创建一个文档并立即刷新索引，使其可见：

```sh
PUT /test/_doc/1?refresh
{"test": "test"}
PUT /test/_doc/2?refresh=true
{"test": "test"}
```

这些将创建一个文档而不做任何使搜索可见的内容：

```sh
PUT /test/_doc/3
{"test": "test"}
PUT /test/_doc/4?refresh=false
{"test": "test"}
```

这将创建一个文档并等待它对搜索可见：

```sh
PUT /test/_doc/4?refresh=wait_for
{"test": "test"}
```
