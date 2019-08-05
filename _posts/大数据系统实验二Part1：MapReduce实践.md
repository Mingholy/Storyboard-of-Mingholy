---
title: 大数据系统实验二Part1：MapReduce实践
date: 2016-05-08 23:00
cover: '7.jpg'
category: "Bigdata"
tags:
    - Java
    - Hadoop
---
## Hadoop实现一个会话统计的MapReduce实例

### Mapper
输入：  
`<source> <destination> <time>`  
可以表示一次电话或者网络访问会话。要求统计不同发起方到不同目标的会话次数和平均时间。
基本的思想是：
1. 对于Mapper，输入是的value是每一行的三个值，通过合法性检验后，作为一个字符串形成key。
2. 如果遇到不合法的输入，比如时间那一项不是个`Double`甚至根本不是个数而是个字符串，这一条数据就要丢弃，继续读取下一条。
3. Mapper的输出格式是`<Text, DoubleWritable>`，它们是Hadoop内建的类型，在存储内容上可以理解为`String`和`Double`。输出的时候是以`<source> <destination>`这一整个字符串作为输出的key，以表示时间的`Double`作为输出的value。
<!--more-->
{% codeblock lang:java %}
    //This is the Mapper class
    public static class HitMapper
        extends Mapper<Object, Text, Text, DoubleWritable>{

        private Text hit = new Text();
        private final static DoubleWritable time = new DoubleWritable();

        public void map(Object key, Text value, Context context )
            throws IOException, InterruptedException {

            StringTokenizer itr = new StringTokenizer(value.toString());

            while (itr.countTokens() >= 2){
                if(checkTime(value.toString())) {
                    //Set hit  as a string of source and destination
                    hit.set(itr.nextToken()+" "+itr.nextToken());
                    time.set(Double.valueOf(itr.nextToken()));
                    context.write(hit, time);
                } else {

                    /**
                     * Discard invalid values
                     * Important!
                     * Here must be 'break', for if the input line is invalid, the loop will break
                     * then read another line, otherwise the whole while-block will fall in infinite loop
                     */
                    break;
                }
            }
        }

       /**
        * Check whether the input line is valid.
        * Invalid inputs are like:
        * source
        *   destination
        *       12345
        * source destination      
        * source destination abcdefg
        */
        public boolean checkTime(String str) {

            String[] args;
            boolean result = false;
            Pattern p =Pattern.compile("\\s+");
            args = p.split(str);

            //check arg[0] arg[1]
            if(args[0] == "" || args[1] == "") {
                return result;
            }

            //check arg[2] to be a double as time
            try {
                if (Double.valueOf(args[2])  >= 0) {
                    result = true;
                } else {
                    result = false;
                }
                return result;
            } catch (Exception e) {
                result = false;
                return result;
            }
        }
    }
{% endcodeblock %}

这里的`StringTokenizer`是一个读取一行由分隔符如`\t\s\n`分隔开的参数时非常有用的一个类。它有若干方法，比如可以取到两个分隔符之间的内容，计数该输入字符串有多少个分隔符，判断剩余的字符串是否还存在分隔符等。  
这里使用了`countTokens()`方法简单判断了一下参数数量是否合法。  
`checkTime()`方法是详细判断参数内容是否合法的方法。具体判断的情况代码中注释已经给出。三个参数包括空值在内的类型不匹配都不合法。  

### Combiner与Shuffle
这里并没有用到Combiner，但是需要记录的一点是，Combiner的作用有一点点向负载均衡靠拢，有一种情况是，假如某个节点经过Shuffle后获得的结果非常非常多，而其他节点又非常少，这样就会造成整个Map Reduce的效率十分低下。有时候这些结果并不是一定需要放在Reducer来处理的，换句话说我们可以在Mapper端对输出进行一定的处理，比如进行一下归纳统计，输出较少的有用的值，而不是把未经处理的原始数据交给Reducer来处理。这样可以提高整个过程的效率。
Combiner实际上扩展实现了Reducer，只是它的输入是Mapper的输出，它的输出是Reducer的输入，有一点点中间人的感觉，进行了一下中间处理。

### Reducer
经过Shuffle后传递给Reducer的是`<Text, Iterable<DoubleWritable>>`这样的值。当然Reducer的泛型仍然是`<Text, DoubleWritable>`只是其中的方法要处理的是一个迭代组，即value的列表。  
统计平均时长就可以把该key下的每一个value加起来，同时设置一个计数器统计有多少个value，最后平均值也容易求。


### 总结
最重要的一点是：类型。在java实践中类的概念必须根深蒂固，时时刻刻要想到类型的对应，类方法的定义和使用。所有的东西都是对象，所有的对象都属于某个类。所谓实例化就是在需要的时候创建类的一个具体对象。

下面是完整代码：
{% codeblock lang:java %}
/**
*@author Mingholy
*/

import java.io.*;
import java.util.StringTokenizer;
import java.util.regex.*;
import java.lang.Double;
import java.lang.Integer;
import java.text.DecimalFormat;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.DoubleWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.mapred.TextInputFormat;
import org.apache.hadoop.mapred.TextOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;

public class Hw2Part1 {

    //This is the Mapper class
    public static class HitMapper
        extends Mapper<Object, Text, Text, DoubleWritable>{

        private Text hit = new Text();
        private final static DoubleWritable time = new DoubleWritable();

        public void map(Object key, Text value, Context context )
            throws IOException, InterruptedException {

            StringTokenizer itr = new StringTokenizer(value.toString());

            while (itr.countTokens() >= 2){
                if(checkTime(value.toString())) {
                    //Set hit  as a string of source and destination
                    hit.set(itr.nextToken()+" "+itr.nextToken());
                    time.set(Double.valueOf(itr.nextToken()));
                    context.write(hit, time);
                } else {

                    /**
                     * Discard invalid values
                     * Important!
                     * Here must be 'break', for if the input line is invalid, the loop will break
                     * then read another line, otherwise the whole while-block will fall in infinite loop
                     */
                    break;
                }
            }
        }

       /**
        * Check whether the input line is valid.
        * Invalid inputs are like:
        * source
        *   destination
        *       12345
        * source destination      
        * source destination abcdefg
        */
        public boolean checkTime(String str) {

            String[] args;
            boolean result = false;
            Pattern p =Pattern.compile("\\s+");
            args = p.split(str);

            //check arg[0] arg[1]
            if(args[0] == "" || args[1] == "") {
                return result;
            }

            //check arg[2] to be a double as time
            try {
                if (Double.valueOf(args[2])  >= 0) {
                    result = true;
                } else {
                    result = false;
                }
                return result;
            } catch (Exception e) {
                result = false;
                return result;
            }
        }
    }
    //This is the Reducer class
    public static class HitReducer
        extends Reducer<Text, DoubleWritable, Text, Text> {

        private Text result_key = new Text();
        private Text result_value = new Text();

        public void reduce(Text key, Iterable<DoubleWritable> values, Context context)
            throws IOException, InterruptedException {
                double time = 0.000;
                double avg_time;
                int count = 0;
                for (DoubleWritable val : values) {
                    time += val.get();
                    count++;
                }
                avg_time = time/count;
                DecimalFormat df = new DecimalFormat("0.000");
                String avg = df.format(avg_time);

                result_key.set(key);

                result_value.set(Integer.toString(count).trim() + " " + avg);

                context.write(result_key, result_value);
        }
    }

    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        String[] arguments = new GenericOptionsParser(conf, args).getRemainingArgs();

        if(arguments.length < 2){
            System.err.println("Usage: hitcount <in> <out>");
            System.exit(2);
        }

        Job job = Job.getInstance(conf, "hit count");

        job.setJarByClass(Hw2Part1.class);

        job.setMapperClass(HitMapper.class);
        job.setReducerClass(HitReducer.class);

        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(DoubleWritable.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(Text.class);

        // add the input paths as given by command line
        for (int i = 0; i < arguments.length - 1; ++i) {
             FileInputFormat.addInputPath(job, new Path(arguments[i]));
        }

        // add the output path as given by the command line
        FileOutputFormat.setOutputPath(job,new Path(arguments[arguments.length - 1]));

        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}
{% endcodeblock %}
