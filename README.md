# Community Detection Analysis of derStandard.at User Interactions

![R](https://img.shields.io/badge/R-565656?logo=r&logoColor=white)
![RStudio IDE](https://img.shields.io/badge/RStudio-565656?logo=rstudioide&logoColor=white)
![Rdplyr](https://img.shields.io/badge/dplyr-%E2%9C%94-blue?logo=librariesdotio&logoColor=white)
![igraph](https://img.shields.io/badge/igraph-%E2%9C%94-blue?logo=librariesdotio&logoColor=white)

## Project Overview

This project focuses on uncovering social structures and user behavior patterns within **derStandard.at** by applying advanced network analysis techniques.  
Through the use of multiple community detection algorithms and efficient data processing pipelines, it enhances content targeting and engagement strategies.

## Key Contributions

- **Led Community Detection Analysis**: Conducted in-depth analysis of user interactions to identify key communities and social trends.
- **Applied Multiple Algorithms**: Utilized six different network analysis algorithms to detect and map user communities, uncovering important engagement and behavior patterns.
- **Enhanced Insights**: Improved understanding of active user groups, supporting better content targeting and engagement strategies.
- **Automated Data Pipelines**: Developed and automated data workflows using **R**, **igraph**, and **dplyr** to process large-scale datasets reliably and efficiently.

## Architecture
```text
+---------------------------+      +---------------------------+     +---------------------------+     +---------------------------+      +---------------------------+
|  User Interaction Dataset | -->  |    Data Cleaning (dplyr)  | --> |  Graph Creation (igraph)  | --> |  Community Detection      | -->  |    Insights & Reports     |
|                           |      |                           |     |                           |     | (6 Network Algorithms)    |      |                           |
+---------------------------+      +---------------------------+     +---------------------------+     +---------------------------+      +---------------------------+
```

## Business Impact

- **Optimized Content Strategy**: Provided deeper insights into active user communities, enabling more targeted and engaging content.
- **Supported Data-Driven Decision Making**: Empowered marketing and content teams with actionable intelligence to optimize user engagement.

## Results
* Here are some output plots based on small dataset, that show detected communities using the menttioned algorithms. For more information, you can have a look at the [report](https://www.overleaf.com/read/fgqqbqstwypq) and [R Markdown document](doc/Community-Detection_smal_dataset.md).


<table>
<tr> <td>
<img src="doc/Community-Detection_smal_dataset_files/figure-gfm/unnamed-chunk-10-1.png" alt=“Binary”>
</td>
<td> <img src="doc/Community-Detection_smal_dataset_files/figure-gfm/unnamed-chunk-10-2.png" alt="Binary "> </td>
 <td> <img src="doc/Community-Detection_smal_dataset_files/figure-gfm/unnamed-chunk-10-3.png" alt="Binary "> </td>
</tr>
 <tr> <td>
<img src="doc/Community-Detection_smal_dataset_files/figure-gfm/unnamed-chunk-10-4.png" alt=“Binary”>
</td>
<td> <img src="doc/Community-Detection_smal_dataset_files/figure-gfm/unnamed-chunk-10-5.png" alt="Binary "> </td>
 <td> <img src="doc/Community-Detection_smal_dataset_files/figure-gfm/unnamed-chunk-10-6.png" alt="Binary "> </td>
</tr>
</table>

* For channel categories with 100- 500 nodes, the following table shows comparative scores by using the above mentioned six algorithms:


<img src="fig/results_100-500.png" alt="Binary "> 

## Final Report Overleaf

https://www.overleaf.com/read/fgqqbqstwypq
