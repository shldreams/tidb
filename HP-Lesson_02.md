

1.资源配置:

| role   | cpu  | memory | disk |
| ------ | ---- | ------ | ---- |
| tidb*1 | 48c  | 20G    | nvme |
| pd*3   | 48c  | 20G    | nvme |
| tikv*9 | 48c  | 20G    | nvme |

![img](file:///private/var/folders/v1/2qb0wgsd12l48kndqc9x05mc0000gn/T/WizNote/e54bf1b7-7c6d-407d-8545-e3a5c893be86/index_files/65358808.png)



 2.参数配置:

```
server_configs:
  tidb: {}
  tikv:
    readpool.coprocessor.use-unified-pool: true
    readpool.storage.use-unified-pool: true
    readpool.unified.max-thread-count: 32
    storage.block-cache.capacity: 20GB
  pd:
    replication.enable-placement-rules: true
    replication.location-labels:
    - host
```

3.sysbench select

![img](file:///private/var/folders/v1/2qb0wgsd12l48kndqc9x05mc0000gn/T/WizNote/e54bf1b7-7c6d-407d-8545-e3a5c893be86/index_files/65814393.png)

![img](file:///private/var/folders/v1/2qb0wgsd12l48kndqc9x05mc0000gn/T/WizNote/e54bf1b7-7c6d-407d-8545-e3a5c893be86/index_files/62078599.png)

![img](file:///private/var/folders/v1/2qb0wgsd12l48kndqc9x05mc0000gn/T/WizNote/e54bf1b7-7c6d-407d-8545-e3a5c893be86/index_files/62158880.png)

![img](file:///private/var/folders/v1/2qb0wgsd12l48kndqc9x05mc0000gn/T/WizNote/e54bf1b7-7c6d-407d-8545-e3a5c893be86/index_files/62217620.png)

![img](file:///private/var/folders/v1/2qb0wgsd12l48kndqc9x05mc0000gn/T/WizNote/e54bf1b7-7c6d-407d-8545-e3a5c893be86/index_files/62233296.png)

update

![img](file:///private/var/folders/v1/2qb0wgsd12l48kndqc9x05mc0000gn/T/WizNote/e54bf1b7-7c6d-407d-8545-e3a5c893be86/index_files/66597250.png)

![img](file:///private/var/folders/v1/2qb0wgsd12l48kndqc9x05mc0000gn/T/WizNote/e54bf1b7-7c6d-407d-8545-e3a5c893be86/index_files/63135324.png)

![img](file:///private/var/folders/v1/2qb0wgsd12l48kndqc9x05mc0000gn/T/WizNote/e54bf1b7-7c6d-407d-8545-e3a5c893be86/index_files/63202372.png)

![img](file:///private/var/folders/v1/2qb0wgsd12l48kndqc9x05mc0000gn/T/WizNote/e54bf1b7-7c6d-407d-8545-e3a5c893be86/index_files/63215752.png)

![img](file:///private/var/folders/v1/2qb0wgsd12l48kndqc9x05mc0000gn/T/WizNote/e54bf1b7-7c6d-407d-8545-e3a5c893be86/index_files/63330316.png)

read_only

![img](file:///private/var/folders/v1/2qb0wgsd12l48kndqc9x05mc0000gn/T/WizNote/e54bf1b7-7c6d-407d-8545-e3a5c893be86/index_files/69993219.png)

![img](file:///private/var/folders/v1/2qb0wgsd12l48kndqc9x05mc0000gn/T/WizNote/e54bf1b7-7c6d-407d-8545-e3a5c893be86/index_files/64096158.png)

![img](file:///private/var/folders/v1/2qb0wgsd12l48kndqc9x05mc0000gn/T/WizNote/e54bf1b7-7c6d-407d-8545-e3a5c893be86/index_files/64144742.png)

![img](file:///private/var/folders/v1/2qb0wgsd12l48kndqc9x05mc0000gn/T/WizNote/e54bf1b7-7c6d-407d-8545-e3a5c893be86/index_files/64159440.png)

![img](file:///private/var/folders/v1/2qb0wgsd12l48kndqc9x05mc0000gn/T/WizNote/e54bf1b7-7c6d-407d-8545-e3a5c893be86/index_files/64187547.png)

go-ycsb

workloada

![img](file:///private/var/folders/v1/2qb0wgsd12l48kndqc9x05mc0000gn/T/WizNote/e54bf1b7-7c6d-407d-8545-e3a5c893be86/index_files/65262932.png)

![img](file:///private/var/folders/v1/2qb0wgsd12l48kndqc9x05mc0000gn/T/WizNote/e54bf1b7-7c6d-407d-8545-e3a5c893be86/index_files/65302507.png)

![img](file:///private/var/folders/v1/2qb0wgsd12l48kndqc9x05mc0000gn/T/WizNote/e54bf1b7-7c6d-407d-8545-e3a5c893be86/index_files/65323755.png)

![img](file:///private/var/folders/v1/2qb0wgsd12l48kndqc9x05mc0000gn/T/WizNote/e54bf1b7-7c6d-407d-8545-e3a5c893be86/index_files/65334768.png)

![img](file:///private/var/folders/v1/2qb0wgsd12l48kndqc9x05mc0000gn/T/WizNote/e54bf1b7-7c6d-407d-8545-e3a5c893be86/index_files/65357276.png)

workloadb

![img](file:///private/var/folders/v1/2qb0wgsd12l48kndqc9x05mc0000gn/T/WizNote/e54bf1b7-7c6d-407d-8545-e3a5c893be86/index_files/65738995.png)

workloadc

![img](file:///private/var/folders/v1/2qb0wgsd12l48kndqc9x05mc0000gn/T/WizNote/e54bf1b7-7c6d-407d-8545-e3a5c893be86/index_files/66003198.png)

workloadd

![img](file:///private/var/folders/v1/2qb0wgsd12l48kndqc9x05mc0000gn/T/WizNote/e54bf1b7-7c6d-407d-8545-e3a5c893be86/index_files/66245053.png)

workloade

![img](file:///private/var/folders/v1/2qb0wgsd12l48kndqc9x05mc0000gn/T/WizNote/e54bf1b7-7c6d-407d-8545-e3a5c893be86/index_files/67369779.png)

![img](file:///private/var/folders/v1/2qb0wgsd12l48kndqc9x05mc0000gn/T/WizNote/e54bf1b7-7c6d-407d-8545-e3a5c893be86/index_files/67416343.png)

![img](file:///private/var/folders/v1/2qb0wgsd12l48kndqc9x05mc0000gn/T/WizNote/e54bf1b7-7c6d-407d-8545-e3a5c893be86/index_files/67443066.png)

![img](file:///private/var/folders/v1/2qb0wgsd12l48kndqc9x05mc0000gn/T/WizNote/e54bf1b7-7c6d-407d-8545-e3a5c893be86/index_files/67453292.png)

![img](file:///private/var/folders/v1/2qb0wgsd12l48kndqc9x05mc0000gn/T/WizNote/e54bf1b7-7c6d-407d-8545-e3a5c893be86/index_files/67479764.png)