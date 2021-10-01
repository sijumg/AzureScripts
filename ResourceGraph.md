## Resource Graph Explorer to find Public IP which are not associated.

```
Resources
| where type contains 'publicIPAddresses' and isnotempty(properties.ipAddress)
| project properties.ipConfiguration.id, properties.ipAddress, name
| limit 100
```


![image](https://user-images.githubusercontent.com/50022305/135696051-17dfcfd7-3704-400a-99a9-c968b65a680e.png)


