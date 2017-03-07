### tensorflow移动端处理图片

#### 可参考博客

[博客零，忽略docker部分，用bazel训练一个pc可用的图片分类模型](https://petewarden.com/2016/02/28/tensorflow-for-poets/)

[博客一，将上面的模型处理为移动端可用的](https://petewarden.com/2016/09/27/tensorflow-for-mobile-poets/)

[博客二，可参考](http://howldb.com/p/tensorflow-for-mobile-poets-0k6yp5)

[博客三，使用的是python脚本训练模型，可能需要翻墙](https://codelabs.developers.google.com/codelabs/tensorflow-for-poets/index.html#4)



#### python脚本（python3下）

* spider.py从百度图片爬相应类别的图片，到config.py中编写你需要的类别。注意爬之前先到百度图片上按照类名搜一下，对同一类的图片不同的名字爬取的效果不同
* deal_pics.py处理下载后的图片，因为tf处理过程中如果图片格式错误或者无法打开会终止处理，所以需要先把不合法的图删除。自己看一下改一下或者自己写一下。

#### 一般需要的命令（没有检验，参考上述博客）

```
########################
# build 图片分类工具，在tensorflow根目录下
bazel build -c opt --copt=-mavx tensorflow/examples/image_retraining:retrain
# 运行图片处理
bazel-bin/tensorflow/examples/image_retraining/retrain \
--bottleneck_dir=/tf_files/bottlenecks \
--model_dir=/tf_files/inception \
--output_graph=/tf_files/retrained_graph.pb \ # 训练后得到的模型位置和名称
--output_labels=/tf_files/retrained_labels.txt \ # 模型的lable，都有啥类别（处理后得到的是索引，然后到这里相应的位置对应类别
--image_dir /tf_files/flower_photos # 待训练的图片的位置，flower_photos是总目录，下面按照类别还有子目录，类别名就是子目录名，子目录下是相应的图片，
########################
# pc上使用分类模型
# build工具
bazel build tensorflow/examples/label_image:label_image
#分类
bazel-bin/tensorflow/examples/label_image/label_image \
--graph=/tf_files/retrained_graph.pb \
--labels=/tf_files/retrained_labels.txt \
--output_layer=final_result \
--image=/tf_files/flower_photos/daisy/21652746_cc379e0eea_m.jpg #待分类图片位置

#########################
# 移动端迁移，为了保证tf在移动端较小，因此部分pc上可用的功能并不在移动端的动态链接库上，一次模型不能直接使用，需要预处理
# 移动端需要的动态链接库可以到tf的github上直接下载，现在已经有了
# 预处理工具
bazel build tensorflow/python/tools:optimize_for_inference
# 处理
bazel-bin/tensorflow/python/tools/optimize_for_inference \
--input=/tf_files/retrained_graph.pb \
--output=/tf_files/optimized_graph.pb \ #处理后的模型名
--input_names=Mul \
--output_names=final_result

# ios请参考0和1两篇博客，以上命令都可以在上面找到
# ios上博客https://petewarden.com/2016/09/27/tensorflow-for-mobile-poets/
```

