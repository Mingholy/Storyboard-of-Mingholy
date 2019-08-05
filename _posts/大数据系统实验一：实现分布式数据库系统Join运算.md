---
title: 大数据系统实验一：实现分布式数据库系统Join运算
date: 2016-04-08 23:00
cover: '7.jpg'
category: "Bigdata"
tags:
    - Java
---
##大数据系统与大规模数据分析 第一次作业
> 了解HDFS和Hbase的基本操作，能够进行读写。并自己实现对数据库表的JOIN操作

作业要求输入参数：
{% codeblock lang:bash %}
java Hw1Grp1 R=/hw1/join\_1.tbl S=/hw1/join\_2.tbl join:R0=S0 res:R1,R2,S1,S2
{% endcodeblock %}
参数所表示的意思就是，输入两个表R和S的路径，join key，以及需要显示的结果列R1，R2，S1，S2。

因为我也没有写过java，很多东西都是霰弹枪式编程，以学习为主。基本思路都体现在步骤里了。

<!--more-->

### 参数处理
由于参数比较长，觉得有必要存成一个对象，就声明了一个类，把两个表的路径存成字符串，两个join key存成整数，其实就是它在表中是第几列；把结果列暂时存成一个数组。
`formatArgs`方法就是用来解析参数的。  
首先按"="切分表的路径，只存路径部分，作为传给读取方法的参数。  
然后对参数数组中的第二个元素，即字符串"join:R0=S0"，按"[=:]"切分，再分别匹配一个整数，就可以得到两个join key。
最后输出列的数组处理略多一点，因为要识别表和列。这里仅仅是存储字符串，识别表名和列放在输出方法中。

### 读取与输出
从HDFS中按行读出表中内容，存成字符串，再根据TPCH表的标准按照"|"切分字符串，这样每一个字符串就变成一个字符型数组。每个数组作为一个对象，将一张表存成一个ArrayList。
输出比较繁琐，因为JOIN操作可能出现不同的记录Column Family和Column Name都相同的情况，这样在put到Hbase时就会出现后put的覆盖之前的记录的情况。于是需要对重复记录重命名成R.1这样的形式。
思路是每输出一条记录，就将它的join key与前一条记录比较，如果相同就重命名该记录相应的列名。
输出的过程是对于每一条记录，循环输出参数中给出的所有列，这时需要取参数中的结果列数组，正则识别每一个元素的列名和列号。
这里需要注意的是，由于join key都是字符串，所以需要用`compareTo()`方法来确定join key的异同。不能用`==`。

### 排序
排序这里自定义了一个比较器。这是为了在一个比较方法中，既可以按照整数大小排序，也可以按照字符串的规则处理。

### 归并
虽然自定义了比较器，但是我不知道如何把这个比较器用到其他方法中。所以这里又写了一个公用的比较方法...

虽然这是一个比较简单的作业，但是其中很多细节还是需要做好才能满分通过。

{% codeblock lang:java %}
/**
*@author Minghao Li 201528015329005
*/

import java.io.*;
import java.math.BigDecimal;
import java.util.regex.*;
import java.util.Arrays;
import java.util.ArrayList;
import java.util.Comparator;
import java.util.Collections;
import java.net.URI;
import java.net.URISyntaxException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FSDataInputStream;
import org.apache.hadoop.fs.FSDataOutputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IOUtils;

import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.HColumnDescriptor;
import org.apache.hadoop.hbase.HTableDescriptor;
import org.apache.hadoop.hbase.MasterNotRunningException;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.ZooKeeperConnectionException;
import org.apache.hadoop.hbase.client.HBaseAdmin;
import org.apache.hadoop.hbase.client.HTable;
import org.apache.hadoop.hbase.client.Put;

import org.apache.log4j.*;

/**
* This is a class of Homework 1.
*<p>Main function: <br>
*Read in two tables and try to JOIN them using the arguments indicated.
*/
public class Hw1Grp1{
    /**
        * Read in a table from HDFS.
        *@param args  table name
        *@return an arraylist object storing a table
        *@exception IOException, URISyntaxException
     */
    public static ArrayList hdfsRead(String args) throws IOException, URISyntaxException{

        String file= "hdfs://localhost:9000" + args;

        Configuration conf = new Configuration();
        FileSystem fs = FileSystem.get(URI.create(file), conf);
        Path path = new Path(file);
        FSDataInputStream in_stream = fs.open(path);
        BufferedReader in = new BufferedReader(new InputStreamReader(in_stream));

        /** create a variable size  array to save the table record as a string array each */
        String s;
        String[] str;
        ArrayList table = new ArrayList();
        Pattern tab = Pattern.compile("[|]+");

        while ((s=in.readLine())!=null) {
            str = tab.split(s);
            table.add(str);
        }

        in.close();
        fs.close();

        return table;
    }

    /**
    *Commit result to HBase
    *@param result   JOIN result table to wite into HBase
    *@param resCol   columns that input arguments required to save in result
    *@param joinKeyR   joinKey index in table R, use this integer to caculate the position of result column in S
    *@param sizeR   column counts of table R, use this integer to caculate the position of result column in S
    *@exception   MasterNotRunningException, ZooKeeperConnectionException, IOException
    */
    public static void hbaseWrite(ArrayList result, ArrayList<String> resCol, int joinKeyR, int sizeR) throws MasterNotRunningException, ZooKeeperConnectionException, IOException {

    Logger.getRootLogger().setLevel(Level.WARN);

    /** create table descriptor */
    String tableName= "Result";
    HTableDescriptor htd = new HTableDescriptor(TableName.valueOf(tableName));

    /** create column descriptor */
    HColumnDescriptor cf = new HColumnDescriptor("res");
    htd.addFamily(cf);

    /** configure HBase */
    Configuration configuration = HBaseConfiguration.create();
    HBaseAdmin hAdmin = new HBaseAdmin(configuration);

    /** recreate the table if table 'Result' already exists */
    if (hAdmin.tableExists(tableName)) {
        hAdmin.disableTable(tableName);
        hAdmin.deleteTable(tableName);
        hAdmin.createTable(htd);
        System.out.println("Table "+tableName+" recreated!");
    }
    else {
        hAdmin.createTable(htd);
        System.out.println("table "+tableName+ " created successfully");
    }
    hAdmin.close();

    HTable table = new HTable(configuration,tableName);

    /** parse the result table and put */
    int i = 0, j = 0;
    int column = 0;
    int duplicate = 0;
    String addDuplicate = new String();
    String[] temp,lastRecord;
    Pattern columnNumber = Pattern.compile("\\d+");

    while(j < result.size()) {
        temp = (String[])result.get(j);

        /** maintain the index bound of array while checking duplicate */
        if (j == 0) {
            lastRecord = (String[])result.get(result.size()-1);
        } else {
            lastRecord = (String[])result.get(j-1);
        }
        Put put = new Put(temp[joinKeyR].getBytes());
        i = 0;

        /** check duplicate prepare to modify column name */
        if (temp[joinKeyR].compareTo(lastRecord[joinKeyR]) == 0) {
            duplicate++;
            addDuplicate = "." + Integer.toString(duplicate);
        } else {
            duplicate = 0;
            addDuplicate = "";
        }

        /** for every record in result table, output every column value that the input argument indicated */
        while(i < resCol.size()) {
            Matcher columnMatcher = columnNumber.matcher(resCol.get(i));
                if(resCol.get(i).matches("^R\\d+$")) {
                    while(columnMatcher.find()) {
                        column = Integer.valueOf(columnMatcher.group(0)).intValue();
                    }
                } else {
                    while(columnMatcher.find()) {
                        column = Integer.valueOf(columnMatcher.group(0)).intValue() + sizeR;
                    }
                }
            put.add("res".getBytes(), (resCol.get(i) + addDuplicate).getBytes(), temp[column].getBytes());
            i++;
        }
        j++;
        table.put(put);
    }
    table.close();
    System.out.println("put successfully");
  }

    /**
    *Sort Merge Join
    *<p>main method of this homework
    *@param args input arguments parsed by method formatArgs
    *@exception IOException, URISyntaxException, MasterNotRunningException, ZooKeeperConnectionException
    */
    public static void main(String[] args) throws IOException, URISyntaxException, MasterNotRunningException, ZooKeeperConnectionException {

        Arguments arguments = new  Arguments();
        ArrayList tableR = new ArrayList();
        ArrayList tableS= new ArrayList();
        ArrayList result = new ArrayList();

        arguments = formatArgs(args);

        /** read tables from HDFS */
        tableR = hdfsRead(arguments.fileRPath);
        tableS = hdfsRead(arguments.fileSPath);

        /** sort phase create run */
        tableR  = sort(tableR, arguments.joinKeyR);
        tableS  = sort(tableS, arguments.joinKeyS);

        /** merge phase */
        result = merge(tableR, tableS, arguments.joinKeyR, arguments.joinKeyS);

        /** output results in console */
        String[] record = (String[])tableR.get(1);
        //output(result, arguments.resCol, arguments.joinKeyR, record.length);

        /** output resutls to HBase */
        hbaseWrite(result, arguments.resCol, arguments.joinKeyR, record.length);
    }

    /**
    *format the args
    *@param args raw arguments input
    *@return a string array contains every argument
    */
    public static Arguments formatArgs(String[] args){

        Pattern splitArgs = Pattern.compile("[:,=]+");
        Pattern extractNum = Pattern.compile("\\d+");
        String[] str;
        int i, j = 1;
        Arguments  arguments = new Arguments();

        /** dispatch arguments to attributes of  object arguments*/
        for ( i = 0; i < 4; i++) {
            str = splitArgs.split(args[i]);
            switch(i) {
                case 0:
                    arguments.fileRPath = str[str.length-1];
                    break;
                case 1:
                    arguments.fileSPath = str[str.length-1];
                    break;
                case 2:
                    Matcher matcher = extractNum.matcher(args[2]);
                    while (matcher.find()){
                        if ( arguments.joinKeyR == -1){
                            arguments.joinKeyR = Integer.valueOf(matcher.group(0)).intValue();
                        } else {
                            arguments.joinKeyS = Integer.valueOf(matcher.group(0)).intValue();
                        }
                    }
                    break;
                case 3:
                    while(j < str.length){
                        arguments.resCol.add(str[j]);
                        j++;
                    }
                    break;
            }
        }
        return arguments;
    }

    /**
    *sort the table using indicated join key
    *<p>override default comparator implementation
    *@param targetList  arraylist to sort
    *@param sortField indicates sort by which column
    *@return a sorted table
    */
    public static ArrayList sort (ArrayList targetList, final int sortField){

        Collections.sort(targetList, new Comparator(){
            @Override
            public int compare(Object r1, Object r2){
                int value = 0;
                String[] record1 = (String[]) r1;
                String[] record2 = (String[]) r2;
                String cr1 = record1[sortField].trim();
                String cr2 = record2[sortField].trim();
                String p = "^(-?\\d+)(\\.\\d+)?$";

                try {
                    if (cr1.matches(p) && cr2.matches(p)){
                        BigDecimal cn1 = new BigDecimal(cr1);
                        BigDecimal cn2 = new BigDecimal(cr2);
                        value = (int)cn1.subtract(cn2).doubleValue();
                    } else {
                        value = cr1.compareTo(cr2);
                    }

                } catch (Exception e) {
                    throw new  RuntimeException();
                }
                return value;
            }
        });
        return targetList;
    }

    /**
    *public compare method
    * <p>compare two rows' join keys see if they're equal<br>
    *   if a is greater than b returns positive value
    *@param record1 one row to compare
    *@param record2 another row
    *@param joinKeyR indicates join key of record1
    *@param joinKeyS indicates join key of record2
    *@return a positive integer if a is greater than b returns positive value else negative
    */
    public static int mergeCompare(Object record1, Object record2, int joinKeyR, int joinKeyS){

        int value = 0;
        String[] r1 = (String[]) record1;
        String[] r2 = (String[]) record2;
        String cr1 = r1[joinKeyR].trim();
        String cr2 = r2[joinKeyS].trim();
        String p = "^(-?\\d+)(\\.\\d+)?$";

        try {
            if (cr1.matches(p) && cr2.matches(p)){
                BigDecimal cn1 = new BigDecimal(cr1);
                BigDecimal cn2 = new BigDecimal(cr2);
                value = (int)cn1.subtract(cn2).doubleValue();
            } else {
                value = cr1.compareTo(cr2);
            }

            } catch (Exception e) {
                    throw new  RuntimeException();
            }
        return value;
    }

    /**
    *merge method
    *@return a merged result table in which join operation has been accomplished
    */
    public static ArrayList merge(ArrayList tableR, ArrayList tableS, int joinKeyR, int joinKeyS) {

        int scanR = 0,scanS1 = 0, scanS2 = 0;
        String[] recR = (String[])tableR.get(1);
        String[] recS = (String[])tableS.get(1);
        String[] addResult;
        ArrayList result = new ArrayList();

        while (scanR < tableR.size() && scanS1 < tableS.size()) {
            if (mergeCompare(tableR.get(scanR), tableS.get(scanS1), joinKeyR, joinKeyS) < 0) {
                scanR++;
            } else if (mergeCompare(tableR.get(scanR), tableS.get(scanS1), joinKeyR, joinKeyS) > 0) {
                scanS1++;
            } else {
                scanS2 = scanS1;
                while (scanS2 < tableS.size() && mergeCompare(tableR.get(scanR), tableS.get(scanS2),joinKeyR,joinKeyS) == 0) {
                    addResult = new String[recR.length + recS.length];
                    recR = (String[])tableR.get(scanR);
                    recS = (String[])tableS.get(scanS2);
                    System.arraycopy(recR, 0, addResult, 0, recR.length)       ;
                    System.arraycopy(recS, 0, addResult, recR.length, recS.length);
                    result.add(addResult);
                    scanS2++;
                }
                scanR++;
            }
        }

        return result;
    }

    /**
    *output method
    *<p>output result table to console in test
    *@param sizeR to caculate the offset of columns in table S
    */
    public static void output(ArrayList result, ArrayList<String> resCol, int joinKeyR, int sizeR) {

        int i = 0, j = 0;
        int column = 0;
        String output = new String();
        String[] temp;
        Pattern columnNumber = Pattern.compile("\\d+");

        while(j < result.size()) {
            temp = (String[])result.get(j);
            output = "join key=" + temp[joinKeyR] +", ";
            i = 0;
            while(i < resCol.size()) {
                Matcher columnMatcher = columnNumber.matcher(resCol.get(i));

                    if(resCol.get(i).matches("^R\\d+$")) {
                        while(columnMatcher.find()) {
                            column = Integer.valueOf(columnMatcher.group(0)).intValue();
                         }
                    } else {
                        while(columnMatcher.find()) {
                            column = Integer.valueOf(columnMatcher.group(0)).intValue() + sizeR;
                        }
                    }
                 output = output + resCol.get(i) + "=" + temp[column] + ",";
                 i++;
             }
             j++;
             System.out.println(output);
        }
    }
}

class Arguments {
        String fileRPath = "";
        String fileSPath = "";
        int joinKeyR = -1;
        int joinKeyS = -1;
        ArrayList<String> resCol = new ArrayList<String>();
}
{% endcodeblock %}
