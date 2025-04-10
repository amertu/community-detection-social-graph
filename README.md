# SNA
Empirical study to compare the effectiveness of six community detection algorithms Fast greedy, Label Propagation, Leading Eigenvector, Infomap, Multilevel, and Walktrap on social network data from the Austrian news website [derStandard.at](https://www.derstandard.at/). The analysis focused on evaluating how each algorithm detects underlying user communities based on interaction patterns. Key evaluation criteria included modularity scores, computational efficiency, and the structural coherence of the resulting clusters. This comparison provided valuable insights into the strengths and limitations of each approach for real-world social network analysis and helped identify the most suitable algorithms for uncovering meaningful social structures in online discussion environments.

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
