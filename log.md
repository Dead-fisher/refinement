# Refinement modify

## 2021.02.16

修改`refinement3_copy`文件夹下的目录

#### 改动

**实现了distance restrain**

在source/abnn/rid.py中追加函数

`get_distance(), get_CA_atom(), add_distance_restrain(), `

修改`make_enhc()`函数中`conf plumed`部分，增加实现距离约束部分。

调整position约束为kappa=[2,16,8], 

distance约束固定为bottom = 0.2, kappa = [26,12,8]

**神经网络节点调节**

在source/gen.py中追加函数`get_node_num()`

修改`gen_rid()`增加神经网络的参数自动调节。

调节函数

```python
if cv_dim < 10:
	return 200
else:
	return 200 + 10 * (cv_dim - 10)
```



修改`refinement3_test/mk_rid_refinement.py`文件

#### 改动

Q: mk_rid_refinement.py 中 函数mk_posre函数中，biased_ang 函数是干什么用的？

**控制溶剂个数**

修改后来的模型的盒子大小为-d 0.95，控制 -maxsol num_sol ， 变量num_sol 通过第一个模型的溶剂化后读取top文件得到。

posre文件公用一个

topol文件公用一个

0~5 依次为`初始，DAN1，DAN2，M1，M2，M3`

6,7均为初始



# 2021.02.18

工作目录

```tex
|--R0949.run06
|	|--iter.000000
|		|--00.enhcMD
|			|--000
|				|--traj_comp.xtc****
|				|--conf_init.gro
|			|--001
|			|--002
|			|--...
|	|--iter.000001
|	|--...
|	|--rid.json
|	|--rid.py (working file)
|	|--rwplus
|	|	|--calRWplus
|	|	|--rw.dat
|	|	|--scb.dat
|	|	|--score.rwplus
|	|	|--data
|	|	|	|-- *.xtc
|	|	|	|-- *.pdb
|	|	|	|-- *.topol.tpr
|	|	|--iter.000000
|	|	|	|--selected_pdb
|	|	|	|--cluster_100.pdb
|	|	|	|--solv_ions_100.gro
|	|	|	|--cluster_200.pdb
|	|	|	|--solv_ions_200.gro
​```````````````````````````````````````````````````````````````````````````````````

```

修改mk_rid_refinement.py 增加函数`mk_rwplus()`用于拷贝rwplus相关文件

修改rid.py文件，增加函数`rw_process()`实现了对pdb文件的打分，识别其中能量最低，最终整合成一个pdb文件，由命令：

### `gmx pdb2gmx ... -ignh -heavyh`

修改了控制水分子的方式，通过读取solv.gro文件来获得盒子的长度，为后续的盒子设置参数-box =第一个盒子的边长+0.1. 同时取了最大溶剂数。第一个模型采用参数-d 0.9 后续模型均没有额外设置-d 参数。

离子生成过程增加了盐水的浓度 -conc 0.15

​			遇到问题，离子数相差+-1，因此控制离子数相同。

修改力场的索引(`mk_rid_refinement() line 190`)

修改计算距离的pdb文件索引 (NOT GOOD)  `where is pdb`



2.18

修改温度320 -> 360 grompp.mdp & grompp_restrain.mdp

修复了rw_processs()上的一些路径bug。

**添加了函数，把所有的HIS换成了HSD。**



## 2021.2.20

rw_process后路径的索引，要复原

dis_kappa要跟随大循环改变

iter>0的连接需要修改，conf_init.gro



## TODO

2021.2.21 mu01 运行./gernral_mkres.sh permission denied. 需要给予权限。

/lib/modeling run_res() 函数中 all_task 没有命名而被调用

**all_task_proposed()** 

​		应该是只有一个变量all_task.

**rid.py 文件 497行，会出现#conf_iter000000_000_99.pdb.1#文件干扰**

应更换

**增加-ntmpi 4**

rid.json batch_jobs=True  mu01

rw_process中打分的 [4, 5] 改成 [100, 200]

walker 统一打分。



**refinement3_test. 中有结果。**

**rwplus, TM.sh sh**

**运行。**

温度设置在 



case R0959 walker 6/7贡献很多。

12iter 2 ns

8iter 

### 2.28

看一下RWplus是不是有聚集的情况，有没有明显的分类，对比一下TMscore和RWplus的图；

写一个批量作业的脚本，能够批量完成任务。

### 3.1

尝试GNNRefine，修改结构为GNNRefine之后的。

320 K 1.2 ns 12 iter

```
"bias_nsteps": 300000,
"bias_frame_freq": 300,
"cleanup": true,
```

在平均后加上一次能量最小化。

然后结构对其，

查看一下rid的代码，初始构象是从什么地方拷贝的。



### 3.14

第一批数据训练基本完成

不太好的例子：

```
R0959  -7  running
R0968s2 -2
R0977-D2 -14  rerun
R0986s1  -17  rerun
R0996-D4 -2.3
R0996-D7 -4  rerun
R0999-D3 -3  rerun
R1001 -2
R1004-D2 -12.34  running
```

重新跑一些这些例子。

步长更换为2 ns，再次跑一次

os.chdir()本質上相当于在改变后的文件夹创建一个同样的文件，因此会改变``os.path.abspath(__file__)``的路径，得到一个不存在的文件。

gmx_mpi 版本的使用和测试

**低res的批量测试，需要改的参数：**

**温度可以用，GNN的位置，和target的名字**

**GDTHA批量作图**，**cluster批量分析**



R0959

阈值

聚类阈值调整 目前测试一下聚类到哪里比较好，然后调整过去。

温度  目前是320K，看一下新测试的低res下表现，如果说表现的不是很好，那就仍然在原来的较强的约束下提高温度到360K，但同时也可以尝试一下约束比4.5强但比20弱的情况。

distance约束   需要测试

bias长度    需要测试

mpi 目前还很慢，



![image-20210321120944562](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20210321120944562.png)

而正常的gmx使用openmpi的速度已经到了42.843 ns/day, 显然这个是慢的。

cutoff阈值可以再增大，把**R0986s1**和R1004-D2的这些CV包含进去

biased MD

6 ns

0986s1

cutoff 在两个case都cover住。0.75 0.8

temperature 320 K 340 K 360K

distance restrain 弱到强，四个case。

我们根据一下的方案来进行测试，首先，根据之前产生的两个不好的case，R0986s1, R1004-D2

在mu01上，CV的值是：

R0986s1   [0,1, 2, 3, 4, 5, 6, 7, 8, 9]

R1004-D2   [65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76]

当cutoff = 0.65:

R0986s1   [0, 1, 2, 24, 25, 26, 27, 28, 29, 30, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 80, 81, 88, 89, 90, 91],

R1004-D2   [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 38, 39, 40, 41, 42, 43, 44, 45, 74, 75, 76],

对于  R0986s1  有 [ 3, 4, 5, 6, 7, 8, 9 ] 未加入，

对于 R1004-D2 有 [65, 66, 67, 68, 69, 70, 71, 72, 73] 未加入，

调整cutoff.

0.85  

R0986s1  [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91]

R01004-D2   [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 52, 53, 54, 62, 63, 64, 65, 66, 67, 72, 73, 74, 75, 76]  仍有68,69,70,71未加入

0.86

R0986s1 [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91]

R1004-D2  [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 52, 53, 54, 55, 62, 63, 64, 65, 66, 67, 69, 70, 72, 73, 74, 75, 76]  仍有68未加入

采用cutoff = 0.86

调节 md 6ns

```
"bias_nsteps": 1500000,
 "bias_frame_freq": 1500,
 dt = 0.004
 
R0986s1
R1004-D2
R0974s1
R1002-D1
R0968s2
R0977-D2
R0981-D4
```

温度限制在320K。

测试例子：

R0986s1,  R1004-D2,  R0974s1 R1002-D1 R0968s2 R0977-D2  R0981-D4

_6ns_1,  [20, 6, 8]

_6ns_weak，[4.5,1,8]

设置了一个没有distance restrain 的MD，修改了rid 276行没有增加distance的限制。



