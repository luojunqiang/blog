## Errors and Exceptions

- https://giantfublog.wordpress.com/2015/03/11/errors-and-exceptions/

因为资源获取失败导致的错误是soft-error，例如文件打开失败，应该采用抛出异常的方式进行处理。
因为程序代码问题导致的错误是hard-error，例如下标访问越界，应该立即记录异常位置，终止程序。

readed at 2015-09-04.
