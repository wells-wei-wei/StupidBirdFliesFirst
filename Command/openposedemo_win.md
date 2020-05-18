<!--
 * @Author: your name
 * @Date: 2020-05-18 15:45:49
 * @LastEditTime: 2020-05-18 15:48:12
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: \undefinedc:\Users\conan\Desktop\LongTime\StupidBirdFliesFirst\Command\openposedemo_win.md
--> 
# Openpose的windows版demo运行命令
```
bin/OpenPoseDemo.exe --image_dir C:\Users\conan\Desktop\test --write_json C:\Users\conan\Desktop\pose --model_pose COCO
```
--model_pose 表示使用的模型，默认是使用能输出25个关键点的，COCO表示18个关键点