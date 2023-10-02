

遇到node宕机或者失联太久可能就会导致pod一直处于Terminating状态，这时候使用kubectl delete不一定可以删除，这种状态下pod已经确定已经无法提供服务了。

Kubernetes中提供了grace-period参数，在Pod删除时此选项会起作用，会延迟一定时长才进行删除，缺省未设定的情况下会等待30s之后删除，此处我们指定grace-period为0，表示立刻删除pod。

```shell
kubectl delete pod  [pod name] --force --grace-period=0 -n [namespace]
```

