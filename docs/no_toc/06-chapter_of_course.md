
# Visualize CNVpytor data inside JBrowse



## Learning Objectives

This chapter will cover:  

- How to visualize CNVpytor data in JBrowse.

## Libraries


```
JBrowse version: https://github.com/GMOD/jbrowse/archive/1.16.6-release.tar.gz
 Plugins: 
 - multibigwig (https://github.com/elsiklab/multibigwig )
 - multiscalebigwig (https://github.com/cmdcolin/multiscalebigwig)

 **Note:** The JBrowse development version is required as integration of different jbrowse plugins are needed.
```

## Steps to process CNVpytor data for JBrowse integration

To generate CNVpytor file for JBrowse visualization:
```
cnvpytor -root [pytor files] -export jbrowse [optional argument: output path]

Default export directory name: 
 - For single pytor file input:  jbrowse_[pytor file name]
 - For multiple pytor file input: cnvpytor_jbrowse_export
```
The above command creates all the necessary files that are required to visualize the CNVpytor data. 

To view CNVpytor file using JBrowse, users need to install JBrowse and required plugins (See JBrowse version and plugins section).
`
http://localhost/jbrowse/?data=[export directory] 
`

``` 
# Example usage
cnvpytor -root test.pytor -export jbrowse
http://localhost/jbrowse/?data=jbrowse_test
```

#### Data Description
There are mainly two types of data CNVpytor processes. i.e.; Read depth data from alignment file and SNP data from variant file. Depending on the availability of these two input data, the export function works.

For Read depth data, it exports Raw segmented RD, GC corrected Raw Segmented RD, GC corrected RD partition, CNV calling using RD . All of these Read depth signals are plotted on top of each other on a single horizontal track using color gray, black, red and green respectively.
For SNP data, it exports Binned BAF, Likelihood of the binned BAF signals. These two signals are plotted on top of each other with gray and red color.

<p align="center"></p>
<table>
    <thead>
        <tr>
            <th align="left">Data</th>
            <th align="center">Signal name with color on JBrowse </th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td align="left">Read Depth (RD)</td>
            <td align="left">Raw Segmented RD (Gray) <br>GC Corrected Raw Segmented RD (Black) <br> GC corrected RD partition (Red) <br> CNV call using RD signals (Green)</td>
        </tr>
        <tr>
            <td align="left">SNP</td>
            <td align="left">Binned BAF (Gray) <br> Likelihood of the Binned BAF (Red)</td>
        </tr>
    </tbody>
</table>
<p></p>

CNVpytor does the segmentation for all of the above data based on the user provided bin size. The `multiscalebigwig` provides the option to show the data based on the visualized area on the reference genome, which means if a user visualizes a small region of the genome it shows small bin data and vice versa.           
!


