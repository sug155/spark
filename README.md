### **Basic Spark Installation and Program Execution**

Apache Spark is a fast, in-memory data processing engine with elegant and expressive APIs. Below are the steps to install Apache Spark and execute a basic program.

---

### **1. Prerequisites**

1. **Java 8+**: Spark requires Java 8 or later. Ensure Java is installed by running:
   ```bash
   java -version
   ```

2. **Scala (Optional)**: Spark is written in Scala, but you can run Spark with just Java or Python. If you want to work with Scala, install it as well:
   ```bash
   scala -version
   ```

---

### **2. Installing Apache Spark**

#### **Install Hadoop (Optional)**

Spark can run independently, but it integrates well with Hadoop. If you want to integrate Spark with Hadoop, you need to install Hadoop first. However, for a simple standalone setup, you can skip this step.

#### **Download Apache Spark**

1. Go to the [official Apache Spark website](https://spark.apache.org/downloads.html) and download the latest stable version.
2. Alternatively, use the command line to download the Spark tarball:
   ```bash
   wget https://archive.apache.org/dist/spark/spark-3.4.0/spark-3.4.0-bin-hadoop3.tgz
   ```

#### **Extract the Tarball**
```bash
tar -xvzf spark-3.4.0-bin-hadoop3.tgz
```

#### **Set Up Environment Variables**

1. Open `.bashrc` or `.zshrc` (depending on your shell) to add the environment variables.
   ```bash
   nano ~/.bashrc
   ```

2. Add the following lines at the end of the file:
   ```bash
   export SPARK_HOME=/path/to/spark-3.4.0-bin-hadoop3
   export PATH=$SPARK_HOME/bin:$PATH
   export PYSPARK_PYTHON=python3  # Optional, for Python users
   ```

3. Reload the `.bashrc` file:
   ```bash
   source ~/.bashrc
   ```

---

### **3. Start Spark in Standalone Mode**

Spark can run in standalone mode without requiring a Hadoop cluster.

1. **Start Spark Master**
   ```bash
   start-master.sh
   ```

2. **Start Spark Worker**
   ```bash
   start-worker.sh spark://<master-ip>:7077
   ```

   Replace `<master-ip>` with the IP address of the master node (or `localhost` if running locally).

3. **Verify Spark is Running**

   Open your browser and visit `http://localhost:8080` (or the master’s IP address if running on a cluster). You should see the Spark Web UI.

---

### **4. Run a Simple Spark Program**

Now, let’s execute a basic Spark program. We'll write a simple program in Python, Scala, and Java.

#### **Using PySpark (Python)**
1. Create a Python file `wordcount.py` with the following code:
   ```python
   from pyspark import SparkContext

   # Initialize SparkContext
   sc = SparkContext("local", "WordCount")

   # Create an RDD from a text file
   lines = sc.textFile("input.txt")

   # Perform word count
   words = lines.flatMap(lambda line: line.split(" "))
   word_counts = words.map(lambda word: (word, 1)).reduceByKey(lambda a, b: a + b)

   # Save the output
   word_counts.saveAsTextFile("output")

   # Stop the SparkContext
   sc.stop()
   ```

2. **Run the Python Program**:
   ```bash
   spark-submit wordcount.py
   ```

#### **Using Scala (if preferred)**
1. Create a `WordCount.scala` file:
   ```scala
   import org.apache.spark.SparkContext
   import org.apache.spark.SparkConf

   object WordCount {
     def main(args: Array[String]) {
       val conf = new SparkConf().setAppName("WordCount").setMaster("local")
       val sc = new SparkContext(conf)

       // Read input data
       val lines = sc.textFile("input.txt")

       // Perform word count
       val words = lines.flatMap(line => line.split(" "))
       val wordCounts = words.map(word => (word, 1)).reduceByKey(_ + _)

       // Save the result
       wordCounts.saveAsTextFile("output")

       // Stop SparkContext
       sc.stop()
     }
   }
   ```

2. **Compile and Submit Scala Code**:
   ```bash
   scalac WordCount.scala
   spark-submit WordCount.jar
   ```

#### **Using Java (if preferred)**
1. Create a `WordCount.java` file:
   ```java
   import org.apache.spark.api.java.*;
   import org.apache.spark.api.java.function.Function;
   import org.apache.spark.api.java.function.PairFunction;
   import org.apache.spark.SparkConf;
   import org.apache.spark.SparkContext;
   import scala.Tuple2;

   public class WordCount {
       public static void main(String[] args) {
           SparkConf conf = new SparkConf().setAppName("WordCount").setMaster("local");
           SparkContext sc = new SparkContext(conf);

           // Read input file
           JavaRDD<String> lines = sc.textFile("input.txt");

           // Perform word count
           JavaPairRDD<String, Integer> wordCounts = lines
               .flatMap(line -> Arrays.asList(line.split(" ")).iterator())
               .mapToPair(word -> new Tuple2<>(word, 1))
               .reduceByKey((a, b) -> a + b);

           // Save the output
           wordCounts.saveAsTextFile("output");

           // Stop SparkContext
           sc.stop();
       }
   }
   ```

2. **Compile and Submit Java Code**:
   ```bash
   javac -classpath $(spark-classpath) WordCount.java
   jar -cvf WordCount.jar WordCount.class
   spark-submit --class WordCount WordCount.jar
   ```

---

### **5. View the Results**

After the job is finished, you can check the output stored in the `output` directory:

```bash
hdfs dfs -ls output
hdfs dfs -cat output/part-00000
```

---

### **6. Stop Spark Services**
To stop the Spark services when done:
```bash
stop-worker.sh
stop-master.sh
```



import org.apache.spark.{SparkConf, SparkContext}

object StringConcatenation {
  def main(args: Array[String]) {
    // Set up the SparkConf and SparkContext
    val conf = new SparkConf().setAppName("StringConcatenation").setMaster("local")
    val sc = new SparkContext(conf)

    // Check if the input file path is provided
    if (args.length < 1) {
      println("Usage: StringConcatenation <input_file>")
      System.exit(1)
    }

    // Read the input text file
    val inputFile = args(0)
    val lines = sc.textFile(inputFile)

    // Perform string concatenation for each line
    val concatenatedLines = lines.map(line => {
      val concatenatedString = line + " - Concatenated"
      concatenatedString
    })

    // Collect and print the results
    concatenatedLines.collect().foreach(println)

    // Stop the SparkContext
    sc.stop()
  }
}

---

### **Summary of Commands**

- **Start Spark Master**: `start-master.sh`
- **Start Spark Worker**: `start-worker.sh spark://<master-ip>:7077`
- **Run Python Program**: `spark-submit wordcount.py`
- **Run Scala Program**: `spark-submit WordCount.jar`
- **Run Java Program**: `spark-submit --class WordCount WordCount.jar`
- **Stop Spark**: `stop-master.sh`, `stop-worker.sh`

---

This guide should help you set up Spark and run a simple WordCount program. Let me know if you need further assistance!
