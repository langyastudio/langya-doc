## 数据层次的划分

具体仓库的分层情况需要结合业务场景、数据场景、系统场景进行综合考虑，常见的分层：

- ODS：Operational Data Store，操作数据层

  在结构上其与源系统的增量或者全量数据基本保持一致。它相当于一个数据准备区，同时又承担着基础数据的记录以及历史变化。其主要作用是把基础数据引入到数仓。

- CDM：Common Data Model，公共维度模型层，又细分为 DWD 和 DWS

  它的主要作用是完成数据加工与整合、建立一致性的维度、构建可复用的面向分析和统计的明细事实表以及汇总公共粒度的指标 

  - DWD：Data Warehouse Detail，明细数据层
  - DWS：Data Warehouse Summary，汇总数据层

- ADS：Application Data Service，应用数据层