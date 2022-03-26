---
author: Simon D'Morias
date: '2018-09-13 15:05:14'
draft: false
title: Getting All Table Information
type: post
url: /blog/2018/9/13/getting-all-table-information
tags:
- SQL Server
- T-SQL
---

One thing that bugs me in SQL Server is how hard it is to get information about your tables to analyse usage, indexes and size.

This is a query I wrote several years ago and still use today. Information exported includes:  

* All Index information including columns  
* Compression  
* File Groups  
* Space Used  
* Row Count  
* Index Usage
    
```
SELECT O.object_id, O.name TableName, ISNULL(I.[name],'HEAP') IndexName,   
          i.type_desc [IndexType], ISNULL(SDS.name,NPSDS.[name]) FileGroup,  
          PS.row_count [RowCount], CAST(PS.used_page_count * 8 AS money)/1024 SpaceUsed_MB,   
          CAST(PS.reserved_page_count * 8 AS money)/1024 ReservedSpace_MB, CAST(PS.lob_reserved_page_count * 8 as money)/1024 LOB_SpaceReserved,  
          ISNULL(PAS.name,'Not Partitioned') [PartitionName], ISNULL(PF.name,'Not Partitioned') [PartitionFunction],   
          CASE WHEN PAS.name IS NOT NULL AND RV.[value] IS NULL THEN 'OutOfBounds' ELSE RV.[value] END RangeValue,  
          PF.boundary_value_on_right RightBound, P.data_compression_desc [Compression],  
          S.User_Seeks, S.User_Scans, S.User_Lookups, S.User_Updates,   
          LEFT(IndexColumns.IndexColumns,LEN(IndexColumns.IndexColumns)-1) IndexColumns,  
          LEFT(IndexColumnsIncluded.IndexColumnsIncluded,LEN(IndexColumnsIncluded.IndexColumnsIncluded)-1) IndexColumnsIncluded  
     FROM sys.dm_db_partition_stats PS  
          JOIN sys.objects O ON O.object_id = PS.object_id AND O.type = 'U'  
          LEFT JOIN sys.indexes I ON I.object_id = O.object_id AND PS.index_id = I.index_id  
          LEFT JOIN sys.dm_db_index_usage_stats S ON S.[object_id] = I.[object_id] AND s.index_id = i.index_id AND S.database_id = DB_ID()  
          LEFT JOIN sys.partition_schemes PAS ON I.Data_Space_ID = PAS.data_space_id  
          LEFT JOIN sys.partition_functions PF ON PF.function_id = PAS.Function_ID  
          LEFT JOIN sys.partition_parameters PP ON PP.function_id = PF.function_id  
          LEFT JOIN sys.partitions P     ON P.object_id = O.object_id and P.index_id = i.index_id and p.partition_id = PS.partition_id  
          LEFT JOIN sys.partition_range_values RV ON RV.function_id = PF.function_id AND RV.boundary_id = CASE WHEN ISNULL(PF.boundary_value_on_right,0) = 1 THEN p.partition_number-1 ELSE p.partition_number END  
          LEFT JOIN sys.destination_data_spaces DS ON DS.destination_id = P.Partition_Number AND DS.partition_scheme_id = PAS.data_space_id  
          LEFT JOIN sys.data_spaces SDS ON SDS.data_space_id = DS.data_space_id  
          LEFT JOIN sys.data_spaces NPSDS ON NPSDS.data_space_id = i.data_space_id  
          OUTER APPLY (SELECT SC.name+','  
                    FROM sys.index_columns IC   
                    JOIN sys.columns SC ON SC.[object_id] = IC.[object_id] AND SC.column_id = IC.column_id  
                    WHERE I.index_id = IC.index_id AND IC.[object_id] = O.[object_id] AND IC.is_included_column = 0  
                    FOR XML PATH('')  
                    ) IndexColumns (IndexColumns)  
          OUTER APPLY (SELECT SC.name+','  
                    FROM sys.index_columns IC   
                    JOIN sys.columns SC ON SC.[object_id] = IC.[object_id] AND SC.column_id = IC.column_id  
                    WHERE I.index_id = IC.index_id AND IC.[object_id] = O.[object_id] AND IC.is_included_column = 1  
                    FOR XML PATH('')  
                    ) IndexColumnsIncluded(IndexColumnsIncluded)  
     --WHERE O.[name] = 'Account'  
     ORDER BY O.[name], i.type_desc, i.[name], P.partition_number  
```

An example output is shown here:

















  

    
  
    



      

      
        
          
        
        

        
          
            
          


            


             
![Click image to enlarge](/images/TableInfoOutput.png)

            


          


        
          
        

        
          
          
            

Click image to enlarge


          
        
      
        
      

    
