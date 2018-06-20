---
title: Hadoop常用配置
categories:
 - hadoop
tags:
 - hadoop
 - hdfs
 - spatial
---

# hadoop 配置

* HDFS 远程访问
`dfs.namenode.rpc-address=0.0.0.0:8020`

## HDFS 空间数据库支持
 参考：https://community.hortonworks.com/articles/44319/geo-spatial-queries-with-hive-using-esri-geometry.html    
 https://blog.csdn.net/qianshangding0708/article/details/50381966
 
 1. 获取空间数据自定义插件：    
 共三个jar包，`esri-geometry-api-2.1.0.jar  spatial-sdk-hive-2.0.0.jar  spatial-sdk-json-2.0.0.jar`
    * https://github.com/Esri/spatial-framework-for-hadoop
    * https://github.com/Esri/geometry-api-java/wiki
 1. Jar放入${HIVE_HOME}/auxlib目录,所有节点    
    在${HIVE_HOME}中创建文件夹auxlib，然后将自定义jar文件放入该文件夹中。
    `scp -r /usr/hdp/current/hive-server2/auxlib cluster01.node02:/usr/hdp/current/hive-server2/`
    此方法添加shell链接不需要重启Hive。而且比较便捷。Hive Server需要重启生效。
 1. 或直接将Jar加入${HIVE_HOME}/lib目录    
    此方法 shell链接 和 Hive Server需要重启生效。
 1. 重启hive服务
 1. shell连接hive _以系统hive用户登录_，执行命令创建自定义空间函数 **UDF**
 ``` 
 create function ST_Aggr_ConvexHull as 'com.esri.hadoop.hive.ST_Aggr_ConvexHull';
 create function ST_Aggr_Intersection as 'com.esri.hadoop.hive.ST_Aggr_Intersection';
 create function ST_Aggr_Union as 'com.esri.hadoop.hive.ST_Aggr_Union';
 create function ST_Area as 'com.esri.hadoop.hive.ST_Area';
 create function ST_AsBinary as 'com.esri.hadoop.hive.ST_AsBinary';
 create function ST_AsGeoJson as 'com.esri.hadoop.hive.ST_AsGeoJson';
 create function ST_AsJson as 'com.esri.hadoop.hive.ST_AsJson';
 create function ST_AsShape as 'com.esri.hadoop.hive.ST_AsShape';
 create function ST_AsText as 'com.esri.hadoop.hive.ST_AsText';
 create function ST_Bin as 'com.esri.hadoop.hive.ST_Bin';
 create function ST_BinEnvelope as 'com.esri.hadoop.hive.ST_BinEnvelope';
 create function ST_Boundary as 'com.esri.hadoop.hive.ST_Boundary';
 create function ST_Buffer as 'com.esri.hadoop.hive.ST_Buffer';
 create function ST_Centroid as 'com.esri.hadoop.hive.ST_Centroid';
 create function ST_Contains as 'com.esri.hadoop.hive.ST_Contains';
 create function ST_ConvexHull as 'com.esri.hadoop.hive.ST_ConvexHull';
 create function ST_CoordDim as 'com.esri.hadoop.hive.ST_CoordDim';
 create function ST_Crosses as 'com.esri.hadoop.hive.ST_Crosses';
 create function ST_Difference as 'com.esri.hadoop.hive.ST_Difference';
 create function ST_Dimension as 'com.esri.hadoop.hive.ST_Dimension';
 create function ST_Disjoint as 'com.esri.hadoop.hive.ST_Disjoint';
 create function ST_Distance as 'com.esri.hadoop.hive.ST_Distance';
 create function ST_EndPoint as 'com.esri.hadoop.hive.ST_EndPoint';
 create function ST_Envelope as 'com.esri.hadoop.hive.ST_Envelope';
 create function ST_EnvIntersects as 'com.esri.hadoop.hive.ST_EnvIntersects';
 create function ST_Equals as 'com.esri.hadoop.hive.ST_Equals';
 create function ST_ExteriorRing as 'com.esri.hadoop.hive.ST_ExteriorRing';
 create function ST_GeodesicLengthWGS84 as 'com.esri.hadoop.hive.ST_GeodesicLengthWGS84';
 create function ST_GeomCollection as 'com.esri.hadoop.hive.ST_GeomCollection';
 create function ST_Geometry as 'com.esri.hadoop.hive.ST_Geometry';
 create function ST_GeometryAccessor as 'com.esri.hadoop.hive.ST_GeometryAccessor';
 create function ST_GeometryN as 'com.esri.hadoop.hive.ST_GeometryN';
 create function ST_GeometryProcessing as 'com.esri.hadoop.hive.ST_GeometryProcessing';
 create function ST_GeometryRelational as 'com.esri.hadoop.hive.ST_GeometryRelational';
 create function ST_GeometryType as 'com.esri.hadoop.hive.ST_GeometryType';
 create function ST_GeomFromGeoJson as 'com.esri.hadoop.hive.ST_GeomFromGeoJson';
 create function ST_GeomFromJson as 'com.esri.hadoop.hive.ST_GeomFromJson';
 create function ST_GeomFromShape as 'com.esri.hadoop.hive.ST_GeomFromShape';
 create function ST_GeomFromText as 'com.esri.hadoop.hive.ST_GeomFromText';
 create function ST_GeomFromWKB as 'com.esri.hadoop.hive.ST_GeomFromWKB';
 create function ST_InteriorRingN as 'com.esri.hadoop.hive.ST_InteriorRingN';
 create function ST_Intersection as 'com.esri.hadoop.hive.ST_Intersection';
 create function ST_Intersects as 'com.esri.hadoop.hive.ST_Intersects';
 create function ST_Is3D as 'com.esri.hadoop.hive.ST_Is3D';
 create function ST_IsClosed as 'com.esri.hadoop.hive.ST_IsClosed';
 create function ST_IsEmpty as 'com.esri.hadoop.hive.ST_IsEmpty';
 create function ST_IsMeasured as 'com.esri.hadoop.hive.ST_IsMeasured';
 create function ST_IsRing as 'com.esri.hadoop.hive.ST_IsRing';
 create function ST_IsSimple as 'com.esri.hadoop.hive.ST_IsSimple';
 create function ST_Length as 'com.esri.hadoop.hive.ST_Length';
 create function ST_LineFromWKB as 'com.esri.hadoop.hive.ST_LineFromWKB';
 create function ST_LineString as 'com.esri.hadoop.hive.ST_LineString';
 create function ST_M as 'com.esri.hadoop.hive.ST_M';
 create function ST_MaxM as 'com.esri.hadoop.hive.ST_MaxM';
 create function ST_MaxX as 'com.esri.hadoop.hive.ST_MaxX';
 create function ST_MaxY as 'com.esri.hadoop.hive.ST_MaxY';
 create function ST_MaxZ as 'com.esri.hadoop.hive.ST_MaxZ';
 create function ST_MinM as 'com.esri.hadoop.hive.ST_MinM';
 create function ST_MinX as 'com.esri.hadoop.hive.ST_MinX';
 create function ST_MinY as 'com.esri.hadoop.hive.ST_MinY';
 create function ST_MinZ as 'com.esri.hadoop.hive.ST_MinZ';
 create function ST_MLineFromWKB as 'com.esri.hadoop.hive.ST_MLineFromWKB';
 create function ST_MPointFromWKB as 'com.esri.hadoop.hive.ST_MPointFromWKB';
 create function ST_MPolyFromWKB as 'com.esri.hadoop.hive.ST_MPolyFromWKB';
 create function ST_MultiLineString as 'com.esri.hadoop.hive.ST_MultiLineString';
 create function ST_MultiPoint as 'com.esri.hadoop.hive.ST_MultiPoint';
 create function ST_MultiPolygon as 'com.esri.hadoop.hive.ST_MultiPolygon';
 create function ST_NumGeometries as 'com.esri.hadoop.hive.ST_NumGeometries';
 create function ST_NumInteriorRing as 'com.esri.hadoop.hive.ST_NumInteriorRing';
 create function ST_NumPoints as 'com.esri.hadoop.hive.ST_NumPoints';
 create function ST_Overlaps as 'com.esri.hadoop.hive.ST_Overlaps';
 create function ST_Point as 'com.esri.hadoop.hive.ST_Point';
 create function ST_PointFromWKB as 'com.esri.hadoop.hive.ST_PointFromWKB';
 create function ST_PointN as 'com.esri.hadoop.hive.ST_PointN';
 create function ST_PointZ as 'com.esri.hadoop.hive.ST_PointZ';
 create function ST_PolyFromWKB as 'com.esri.hadoop.hive.ST_PolyFromWKB';
 create function ST_Polygon as 'com.esri.hadoop.hive.ST_Polygon';
 create function ST_Relate as 'com.esri.hadoop.hive.ST_Relate';
 create function ST_SetSRID as 'com.esri.hadoop.hive.ST_SetSRID';
 create function ST_SRID as 'com.esri.hadoop.hive.ST_SRID';
 create function ST_StartPoint as 'com.esri.hadoop.hive.ST_StartPoint';
 create function ST_SymmetricDiff as 'com.esri.hadoop.hive.ST_SymmetricDiff';
 create function ST_Touches as 'com.esri.hadoop.hive.ST_Touches';
 create function ST_Union as 'com.esri.hadoop.hive.ST_Union';
 create function ST_Within as 'com.esri.hadoop.hive.ST_Within';
 show functions;
 ```