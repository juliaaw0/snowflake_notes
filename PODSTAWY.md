
 hybrid architecture in which different layers can be scaled independently of one another. The layer responsible for compute or processing can be scaled independently from the layer responsible for storage. It uses cheap cloud storage to store data in a shared manner while still allowing various compute clusters to act on the shared data simultaneously. The compute clusters, also known as virtual warehouses. They are available in a variety of sizes and they can be started, stopped or removed as required. This hybrid architecture is also known as multi clustered shared data architecture. The shared storage layer of snowflake resides on cheap object cloud storage. Currently, Snowflake supports an AWS S3 storage Azure blob storage or Google Cloud storage. Therefore, Snowflake can take advantage of the disaster, recovery and fault tolerance provided by the underlying cloud platform.
![[Pasted image 20260224231615.png]]
![[Pasted image 20260224231850.png]]
Wirtual warehouses are completely independent from the shared storage, but they all access the same shared data.

Not only that, you can have multiple virtual warehouses running simultaneously, but they can also each be of a different configuration and be dedicated for a specific purpose or group of users. They can be started up and shut down as you need, and you incur costs only when they are active. They can even be resized from a smaller size to a larger size so that you can process a more complex query more quickly and then resize back to a smaller size when additional power is not required. This capability to create as many virtual warehouses of the desired configuration provides massively parallel processing or MPP capability to snowflake while still accessing a single shared data. Therefore, the architecture for Snowflake is often referred to as shared data multi cluster architecture.




![[Pasted image 20260225003003.png]]
![[Pasted image 20260225003024.png]]
![[Pasted image 20260225003029.png]]