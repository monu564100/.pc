PROGRAM 1
Hadoop File Management

mkdir Hadoop
cd Hadoop
vi hello.txt
cat hello.txt
hdfs dfs -mkdir /today
hdfs dfs -put hello.txt /today/data.txt
hdfs dfs -mkdir /input
hdfs dfs -cp /today/data.txt /input/data.txt
hdfs dfs -cat /input/data.txt
hdfs dfs -get /input/data.txt
hdfs dfs -rm /today/data.txt
hdfs dfs -rmdir /today
hdfs dfs -ls /
hdfs dfs -ls /input


PROGRAM 2
MatrixMultiplication.java
import java.io.IOException;
import java.util.HashMap;
import java.util.Map;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.*;
import org.apache.hadoop.mapreduce.*;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class MatrixMultiplication {
    public static class Map extends Mapper<Object, Text, Text, Text> {
        public void map(Object key, Text value, Context context)
                throws IOException, InterruptedException {
            String[] x = value.toString().split(",");
            if (x.length != 4) return;
            String matrix = x[0].trim();
            int r = Integer.parseInt(x[1].trim());
            int c = Integer.parseInt(x[2].trim());
            float v = Float.parseFloat(x[3].trim());
            int rowsA = context.getConfiguration().getInt("rowsA", 2);
            int colsB = context.getConfiguration().getInt("colsB", 2);
            if (matrix.equals("A")) {
                for (int j = 0; j < colsB; j++)
                    context.write(new Text(r + "," + j), new Text("A," + c + "," + v));
            } else if (matrix.equals("B")) {
                for (int i = 0; i < rowsA; i++)
                    context.write(new Text(i + "," + c), new Text("B," + r + "," + v));
            }
        }
    }

    public static class Reduce extends Reducer<Text, Text, Text, FloatWritable> {
        public void reduce(Text key, Iterable<Text> values, Context context)
                throws IOException, InterruptedException {
            Map<Integer, Float> a = new HashMap<>();
            Map<Integer, Float> b = new HashMap<>();
            for (Text value : values) {
                String[] x = value.toString().split(",");
                int k = Integer.parseInt(x[1]);
                float v = Float.parseFloat(x[2]);
                if (x[0].equals("A")) a.put(k, v);
                else b.put(k, v);
            }
            float sum = 0;
            for (Map.Entry<Integer, Float> e : a.entrySet())
                if (b.containsKey(e.getKey())) sum += e.getValue() * b.get(e.getKey());
            context.write(key, new FloatWritable(sum));
        }
    }

    public static void main(String[] args) throws Exception {
        if (args.length != 4) {
            System.err.println("Usage: MatrixMultiplication <input> <output> <rowsA> <colsB>");
            System.exit(1);
        }
        Configuration conf = new Configuration();
        conf.setInt("rowsA", Integer.parseInt(args[2]));
        conf.setInt("colsB", Integer.parseInt(args[3]));
        Job job = Job.getInstance(conf, "Matrix Multiplication");
        job.setJarByClass(MatrixMultiplication.class);
        job.setMapperClass(Map.class);
        job.setReducerClass(Reduce.class);
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(Text.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(FloatWritable.class);
        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}

matrix.txt
A,0,0,1
A,0,1,2
A,1,0,3
A,1,1,4
B,0,0,5
B,0,1,6
B,1,0,7
B,1,1,8

Execution:
hadoop com.sun.tools.javac.Main MatrixMultiplication.java
jar cf matrix.jar MatrixMultiplication*.class
hdfs dfs -mkdir -p /matrix/input
hdfs dfs -put matrix.txt /matrix/input/
hadoop jar matrix.jar MatrixMultiplication /matrix/input /matrix/output 2 2
hdfs dfs -cat /matrix/output/part-r-00000


PROGRAM 3
WeatherMapReduce1.java
import java.io.IOException;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class WeatherMapReduce1 {
    public static class Map extends Mapper<Object, Text, Text, Text> {
        public void map(Object key, Text value, Context context)
                throws IOException, InterruptedException {
            String[] data = value.toString().split(",");
            if (data.length < 2) return;
            String city = data[0].trim();
            double temp = Double.parseDouble(data[1].trim());
            String condition;
            if (temp < 0) condition = "Freezing";
            else if (temp <= 15) condition = "Cold";
            else if (temp <= 25) condition = "Warm";
            else condition = "Hot";
            context.write(new Text(city), new Text(condition));
        }
    }

    public static class Reduce extends Reducer<Text, Text, Text, Text> {
        public void reduce(Text key, Iterable<Text> values, Context context)
                throws IOException, InterruptedException {
            StringBuilder result = new StringBuilder();
            for (Text value : values) {
                if (result.length() > 0) result.append(" ");
                result.append(value.toString());
            }
            context.write(key, new Text(result.toString()));
        }
    }

    public static void main(String[] args) throws Exception {
        if (args.length != 2) {
            System.err.println("Usage: WeatherMapReduce1 <input> <output>");
            System.exit(1);
        }
        Job job = Job.getInstance(new Configuration(), "Weather");
        job.setJarByClass(WeatherMapReduce1.class);
        job.setMapperClass(Map.class);
        job.setReducerClass(Reduce.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(Text.class);
        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}

weather.txt
New York,5
Los Angeles,25
Chicago,-3
Miami,30

Execution:
hadoop com.sun.tools.javac.Main WeatherMapReduce1.java
jar cf weather.jar WeatherMapReduce1*.class
hdfs dfs -mkdir -p /weather/input
hdfs dfs -put weather.txt /weather/input/
hadoop jar weather.jar WeatherMapReduce1 /weather/input /weather/output
hdfs dfs -cat /weather/output/part-r-00000


PROGRAM 4
MovieTags.java
import java.io.IOException;
import java.util.HashSet;
import java.util.Set;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class MovieTags {
    public static class Map extends Mapper<Object, Text, Text, Text> {
        public void map(Object key, Text value, Context context)
                throws IOException, InterruptedException {
            String[] data = value.toString().split(",");
            if (data.length == 4)
                context.write(new Text(data[1].trim()), new Text(data[2].trim()));
        }
    }

    public static class Reduce extends Reducer<Text, Text, Text, Text> {
        public void reduce(Text key, Iterable<Text> values, Context context)
                throws IOException, InterruptedException {
            Set<String> tags = new HashSet<>();
            for (Text value : values) tags.add(value.toString().trim());
            context.write(key, new Text(String.join(", ", tags)));
        }
    }

    public static void main(String[] args) throws Exception {
        if (args.length != 2) {
            System.err.println("Usage: MovieTags <input> <output>");
            System.exit(1);
        }
        Job job = Job.getInstance(new Configuration(), "Movie Tags");
        job.setJarByClass(MovieTags.class);
        job.setMapperClass(Map.class);
        job.setReducerClass(Reduce.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(Text.class);
        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}

tags.csv
1,101,comedy,1234567890
2,101,drama,1234567891
3,102,action,1234567892
4,101,comedy,1234567893
5,102,thriller,1234567894

Execution:
hadoop com.sun.tools.javac.Main MovieTags.java
jar cf movietags.jar MovieTags*.class
hdfs dfs -mkdir -p /movies/input
hdfs dfs -put tags.csv /movies/input/
hadoop jar movietags.jar MovieTags /movies/input /movies/output
hdfs dfs -cat /movies/output/part-r-00000


PROGRAM 5
MongoDB Count, Sort, Limit, Skip and Aggregate

use BDA

db.users.insertMany([
  { name: "Alice", age: 30, gender: "Female", city: "New York" },
  { name: "Bob", age: 25, gender: "Male", city: "Los Angeles" },
  { name: "Charlie", age: 35, gender: "Male", city: "Chicago" },
  { name: "David", age: 28, gender: "Male", city: "San Francisco" },
  { name: "Eve", age: 22, gender: "Female", city: "Boston" },
  { name: "Frank", age: 40, gender: "Male", city: "Houston" },
  { name: "Grace", age: 27, gender: "Female", city: "Miami" },
  { name: "Hannah", age: 29, gender: "Female", city: "Seattle" },
  { name: "Isaac", age: 31, gender: "Male", city: "Denver" },
  { name: "Jack", age: 26, gender: "Male", city: "Dallas" }
])

db.users.countDocuments({})
db.users.countDocuments({ age: { $gt: 25 } })
db.users.find().sort({ age: 1 })
db.users.find().limit(5)
db.users.find().skip(9)
db.users.aggregate([
  { $match: { age: { $gt: 25 } } },
  { $group: { _id: "$gender", count: { $sum: 1 } } },
  { $sort: { count: -1 } },
  { $limit: 5 }
])
db.users.find().sort({ age: -1 }).skip(3).limit(5)


PROGRAM 6
Pig Latin Scripts

employees.txt
101,John,IT,50000
102,Susan,HR,60000
103,David,IT,55000
104,Emma,Finance,75000
105,James,HR,62000
106,Robert,IT,58000

departments.txt
IT,Information Technology
HR,Human Resources
Finance,Finance Department

sort.pig
employees = LOAD 'employees.txt' USING PigStorage(',') AS (id:int, name:chararray, dept:chararray, salary:int);
sorted_employees = ORDER employees BY salary DESC;
DUMP sorted_employees;

group.pig
employees = LOAD 'employees.txt' USING PigStorage(',') AS (id:int, name:chararray, dept:chararray, salary:int);
grouped_employees = GROUP employees BY dept;
DUMP grouped_employees;

join.pig
employees = LOAD 'employees.txt' USING PigStorage(',') AS (id:int, name:chararray, dept:chararray, salary:int);
departments = LOAD 'departments.txt' USING PigStorage(',') AS (dept_code:chararray, dept_name:chararray);
joined_data = JOIN employees BY dept, departments BY dept_code;
projected_data = FOREACH joined_data GENERATE employees::id, employees::name, departments::dept_name, employees::salary;
DUMP projected_data;

project.pig
employees = LOAD 'employees.txt' USING PigStorage(',') AS (id:int, name:chararray, dept:chararray, salary:int);
projected_data = FOREACH employees GENERATE name, salary;
DUMP projected_data;

filter.pig
employees = LOAD 'employees.txt' USING PigStorage(',') AS (id:int, name:chararray, dept:chararray, salary:int);
filtered_employees = FILTER employees BY salary > 55000;
DUMP filtered_employees;

Execution:
pig -x local sort.pig
pig -x local group.pig
pig -x local join.pig
pig -x local project.pig
pig -x local filter.pig


PROGRAM 8
Word Count Using Hadoop and Spark

WordCount.java
import java.io.IOException;
import java.util.StringTokenizer;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class WordCount {
    public static class TokenizerMapper extends Mapper<Object, Text, Text, IntWritable> {
        private final static IntWritable one = new IntWritable(1);
        private Text word = new Text();

        public void map(Object key, Text value, Context context)
                throws IOException, InterruptedException {
            StringTokenizer itr = new StringTokenizer(value.toString());
            while (itr.hasMoreTokens()) {
                word.set(itr.nextToken().replaceAll("[^a-zA-Z0-9]", "").toLowerCase());
                if (!word.toString().isEmpty()) context.write(word, one);
            }
        }
    }

    public static class IntSumReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
        private IntWritable result = new IntWritable();

        public void reduce(Text key, Iterable<IntWritable> values, Context context)
                throws IOException, InterruptedException {
            int sum = 0;
            for (IntWritable val : values) sum += val.get();
            result.set(sum);
            context.write(key, result);
        }
    }

    public static void main(String[] args) throws Exception {
        if (args.length != 2) {
            System.err.println("Usage: WordCount <input> <output>");
            System.exit(1);
        }
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf, "word count");
        job.setJarByClass(WordCount.class);
        job.setMapperClass(TokenizerMapper.class);
        job.setCombinerClass(IntSumReducer.class);
        job.setReducerClass(IntSumReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}

input.txt
Hello world
Hello BDA
Hi Hello

Execution:
hadoop com.sun.tools.javac.Main WordCount.java
jar cf wordcount.jar WordCount*.class
hdfs dfs -mkdir -p /wordcount/input
hdfs dfs -put input.txt /wordcount/input/
hadoop jar wordcount.jar WordCount /wordcount/input /wordcount/output
hdfs dfs -cat /wordcount/output/part-r-00000

wordcount.py
from pyspark import SparkContext

sc = SparkContext("local", "Word Count")
text_file = sc.textFile("file:///home/aindsl504022/input.txt")
word_counts = (
    text_file
    .flatMap(lambda line: line.split())
    .map(lambda word: (word, 1))
    .reduceByKey(lambda a, b: a + b)
)
print(word_counts.collect())
word_counts.saveAsTextFile("file:///home/aindsl504022/output")
sc.stop()

Spark execution:
spark-submit wordcount.py
