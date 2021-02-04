# 升级影响分析

配置合并代码

```shell
def dict_merge(a, b):
    '''Recursively merges dicts. The secondary dict takes precedence.

    For lists, extend the fisrt one with second one, and make sure elements
    in resulting list are unique.
    '''
    if isinstance(b, list):
        result = deepcopy(a)
        result.extend(deepcopy(b))
        uniq_res = []
        for x in result:
            if x not in uniq_res:
                uniq_res.append(x)
        return uniq_res
    if not isinstance(b, dict):
        return deepcopy(b)
    if a is None:
        return deepcopy(b)
    result = deepcopy(a)
    for k, v in six.iteritems(b):
        if k in result and (isinstance(result[k], dict) or
                            isinstance(result[k], list)):
            result[k] = dict_merge(result[k], v)
        else:
            result[k] = deepcopy(v)
    return result
```

