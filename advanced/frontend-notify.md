# 前端消息与弹窗

:::danger
**暴露公共提示组件，可在前端任意位置调用**，例如：

1. 在 `app.js` 中调用
2. 在自定义页面中使用 `parent.msg.xxx` 的形式调用
3. 也可在自定义按钮的返回 JS 中调用
4. [erupt-websocket](/modules/erupt-websocket) 推送时调用

:::

::: info
注：1.12.15 及以上版本支持
:::

## 全局提示组件

```javascript
window.msg.info('This is a normal message');
window.msg.success('This is a normal message');
window.msg.error('This is a normal message');
window.msg.warning('This is a normal message');

// loading
let loading = window.msg.loading('This is a normal message', { nzDuration: 2500 });
// 手动关闭 loading
window.msg.remove(loading.messageId);
```

<img src="https://cdn.nlark.com/yuque/0/2024/png/117735/1724580418261-904bba48-f20c-41eb-a349-e6cfd4025c88.png" width="1680">

## 弹窗提示组件

<img src="https://cdn.nlark.com/yuque/0/2024/png/117735/1724580593997-de4d6b65-9ebd-470f-9404-5582b73902f6.png" width="1680">

```javascript
window.modal.info({
  nzTitle: 'This is a notification message',
  nzContent: '<p>some messages...some messages...</p>',
  nzOnOk: () => console.log('Info OK')
})
window.modal.success({
  nzTitle: 'This is a notification message',
  nzContent: '<p>some messages...some messages...</p>',
  nzOnOk: () => console.log('Info OK')
})
window.modal.error({
  nzTitle: 'This is a notification message',
  nzContent: '<p>some messages...some messages...</p>',
  nzOnOk: () => console.log('Info OK')
})
window.modal.warning({
  nzTitle: 'This is a notification message',
  nzContent: '<p>some messages...some messages...</p>',
  nzOnOk: () => console.log('Info OK')
})
window.modal.confirm({
  nzTitle: 'Are you sure delete this task?',
  nzContent: '<b style="color: red;">Some descriptions</b>',
  nzOkText: 'Yes',
  nzOkType: 'primary',
  nzOkDanger: true,
  nzOnOk: () => console.log('OK'),
  nzCancelText: 'No',
  nzOnCancel: () => console.log('Cancel')
})

// 关闭所有弹窗
window.modal.closeAll();
```

## 通知提示组件

<img src="https://cdn.nlark.com/yuque/0/2024/png/117735/1724580881381-0810d64f-ac2b-421b-88d5-355fb9476ed4.png" width="1680">

```javascript
window.notify.blank('title', 'content', [options]) // 不带图标的提示
window.notify.success('title', 'content', {
  nzPlacement: 'bottom'
})
window.notify.error('title', 'content')
window.notify.info('title', 'content')
window.notify.warning('title', 'content')
```

### options 参数

| 参数 | 说明 | 类型 | 默认值 |
| --- | --- | --- | --- |
| `nzDuration` | 持续时间（毫秒），设为 `0` 时不自动消失 | `number` | `4500` |
| `nzMaxStack` | 同一时间可展示的最大提示数量 | `number` | `8` |
| `nzPauseOnHover` | 鼠标移上时禁止自动移除 | `boolean` | `true` |
| `nzAnimate` | 开关动画效果 | `boolean` | `true` |
| `nzTop` | 消息从顶部弹出时距顶部的位置 | `string` | `24px` |
| `nzBottom` | 消息从底部弹出时距底部的位置 | `string` | `24px` |
| `nzPlacement` | 弹出位置：`topLeft` `topRight` `bottomLeft` `bottomRight` `top` `bottom` | `string` | `topRight` |
| `nzDirection` | 通知的文字方向 | `'ltr' \| 'rtl'` | - |

## 在组件中渲染 HTML

> 1.12.23 及以上版本支持

提示组件、弹窗组件、通知组件的内容参数默认只接收普通文本，如需渲染 HTML，需用 `window.safeHtml` 包裹告知框架标签安全可渲染：

```javascript
window.msg.error(window.safeHtml(`
    提示xxx错误
    <button style="margin-left:20px" onclick="window.location.hash = '/xxx'">查看</button>
`), {
    nzDuration: 5000
})
```

## Java 后端推送（WebSocket）

::: warning 前提条件
使用后端推送功能必须先引入 `erupt-websocket` 模块依赖：

```xml
<dependency>
    <groupId>xyz.erupt</groupId>
    <artifactId>erupt-websocket</artifactId>
</dependency>
```

前端页面需建立 WebSocket 连接（Erupt 框架默认自动连接），否则后端推送无法到达前端。
:::

`FrontendNotifyService` 是一个封装好的工具类，可在 `OperationHandler`、`DataProxy`、`Service` 等后端代码中通过 WebSocket 向当前用户的前端推送各类消息通知。

```java
@Resource
private FrontendNotifyService frontendNotifyService;
```

### 全局消息提示（msg）

```java
frontendNotifyService.msgInfo("操作成功");
frontendNotifyService.msgSuccess("数据已保存");
frontendNotifyService.msgError("提交失败，请重试");
frontendNotifyService.msgWarning("该功能即将下线");

// 带选项：5 秒后自动消失
frontendNotifyService.msgInfo("提示消息", Map.of("nzDuration", 5000));
```

### Loading 指示器

```java
// 显示 loading，返回唯一 ID
String loadingId = frontendNotifyService.msgLoading("正在处理...");

try {
    // ... 执行耗时操作 ...
} finally {
    // 关闭 loading
    frontendNotifyService.msgRemove(loadingId);
}
```

### 弹窗提示（modal）

```java
frontendNotifyService.modalInfo("提示", "这是一条通知消息");
frontendNotifyService.modalSuccess("成功", "操作已完成");
frontendNotifyService.modalError("错误", "发生了错误");
frontendNotifyService.modalWarning("警告", "请确认操作");
frontendNotifyService.modalConfirm("确认", "确定要删除该记录吗？");

// 关闭所有弹窗
frontendNotifyService.modalCloseAll();
```

### 通知面板（notify）

```java
frontendNotifyService.notifyInfo("系统通知", "您有新的待办事项");
frontendNotifyService.notifySuccess("任务完成", "报表已生成");
frontendNotifyService.notifyError("任务失败", "导出超时");
frontendNotifyService.notifyWarning("磁盘空间", "剩余空间不足 10%");
frontendNotifyService.notifyBlank("公告", "系统将于今晚 22:00 维护");

// 带选项：显示在左下角
frontendNotifyService.notifySuccess("完成", "导出成功", Map.of("nzPlacement", "bottomLeft"));
```

### 广播（所有在线用户）

```java
frontendNotifyService.broadcastMsgInfo("系统将于今晚 22:00 进行维护");
frontendNotifyService.broadcastNotifyInfo("系统公告", "新版本已发布，请刷新页面");
```

### 在 OperationHandler 中使用示例

```java
@Slf4j
@Component
public class MyHandler implements OperationHandler<MyEntity, Void> {

    @Resource
    private FrontendNotifyService notify;

    @Override
    public String exec(List<MyEntity> data, Void vo, String[] param) {
        String loadingId = notify.msgLoading("正在同步数据...");
        try {
            // ... 业务逻辑 ...
            notify.msgSuccess("同步完成");
            return "操作成功";
        } catch (Exception e) {
            notify.msgError("同步失败: " + e.getMessage());
            return "操作失败: " + e.getMessage();
        } finally {
            notify.msgRemove(loadingId);
        }
    }
}
```

::: info 安全说明
`FrontendNotifyService` 使用 `GsonFactory.getGson().toJson()` 对所有用户输入进行转义，防止 JavaScript 注入。可安全用于包含特殊字符（引号、换行、HTML 标签等）的消息内容。
:::
