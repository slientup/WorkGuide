#### k8s 客户端

[Kubernetes Python Client使用案例](https://cloud.tencent.com/developer/article/1717549)

[Kubernetes Python Client使用案例 kubeconfig文件版本](https://www.cnblogs.com/zhangb8042/p/11444756.html)

[官方使用文档](https://github.com/kubernetes-client/python)

[所有api查询文档](https://github.com/kubernetes-client/python/blob/master/kubernetes/README.md)

api文档写的非常好 接口都可以点进去看到使用案例
![](https://files.mdnice.com/user/4251/d7dcb088-3aed-4bac-a722-79c772e2f976.png)

代码案例：
```python
from kubernetes import client
from kubernetes.client import api_client
from kubernetes.client.api import core_v1_api

# qa  url和token 都在cat ~/.kube/config里
KUBERNETES_URL = "xxx"
AUTH_TOKEN = "xxx"


class KubernetesClient(object):

    def __init__(self):
        self.api = self.get_core_v1_api()

    def _get_configuration(self):
        configuration = client.Configuration()
        configuration.host = KUBERNETES_URL
        configuration.verify_ssl = False
        configuration.api_key = {"authorization": "Bearer " + AUTH_TOKEN}
        return configuration

    def get_core_v1_api(self):
        """
        获取API的CoreV1Api版本对象
        """
        k8s_client = api_client.ApiClient(configuration=self._get_configuration())
        api = core_v1_api.CoreV1Api(k8s_client)
        return api


    def get_all_namespaces(self):
        """
        获取命名空间列表
        """
        namespace_list = []
        for ns in self.api.list_namespace().items:
            print(ns.metadata.name)
            namespace_list.append(ns.metadata.name)

        return namespace_list
```
