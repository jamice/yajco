# Download #

## Maven ##
YAJCo tool uses Maven for including into Java projects, so **you don't need to download any archives - Maven does that for you**. You only need to specify correct dependency for your project in `pom.xml`.

These are dependencies needed for YAJCo annotation processor to work and generate Beaver parser.
```
<dependencies>
    <dependency>
        <groupId>sk.tuke.yajco</groupId>
        <artifactId>yajco-annotation-processor</artifactId>
        <version>0.5.9</version>
    </dependency>
    <dependency>
        <groupId>sk.tuke.yajco</groupId>
        <artifactId>yajco-beaver-parser-generator-module</artifactId>
        <version>0.5.9</version>
    </dependency>
</dependencies>
```

If you prefer JavaCC parser generator use:
```
    <dependency>
        <groupId>sk.tuke.yajco</groupId>
        <artifactId>yajco-javacc-parser-generator-module</artifactId>
        <version>0.5.9</version>
    </dependency>
```
instead of `yajco-beaver-parser-generator-module`.


## Uber JAR ##
In case you are not _a friend_ with Maven, we have prepared special archived JAR with all dependencies included for you. Downfall is that such code is not actively supported, it can get old over time and not all possibilities and tools are included.
| **Type** | **Download** |
|:---------|:-------------|
| YAJCo annotation processor + Beaver generator 0.5.9| [JAR](https://yajco.googlecode.com/svn/uber-jar/yajco-annotation-processor-beaver-generator-0.5.9.jar) |
| YAJCo annotation processor + JavaCC generator 0.5.9| [JAR](https://yajco.googlecode.com/svn/uber-jar/yajco-annotation-processor-javacc-generator-0.5.9.jar) |

## Source code ##
Source code of all project is accesible on this site in section [Source](https://code.google.com/p/yajco/source/checkout).