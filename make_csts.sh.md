---
title: make_csts.sh
date: 2023-03-02 22:56:04
---

### 需求
对于要突变的蛋白质，需要约束其主链，该脚本用于生成主链限制坐标
### 代码

```bash
#!/bin/sh

pdb=$1

#使用 `wc` 命令统计 PDB 文件的行数，使用 `awk` 命令从输出中提取第一列的值，即行数，将其赋值给 `lines` 变量。
lines=`wc $pdb | awk '{print $1}'`; 

#定义坐标约束函数和类型，并从 PDB 文件中提取用于作为约束基准的 C-α 原子的残基号和链标识，并将它们组合成一个字符串 `root`。
constraint_func="HARMONIC 0 1" # harmonic constraint mean=0 std=1
constraint_type="CoordinateConstraint" #constraint type
root_aa=`awk '$1=="ATOM" && $3=="CA"{print substr($0, 23, 4)}' $pdb | head -1`
root_aa_chain=`awk '$1=="ATOM" && $3=="CA"{print substr($0, 22, 1)}' $pdb | head -1 `
root="CA ${root_aa}${root_aa_chain}" #the atom, relative to which the constraints are computed

#使用 `seq` 命令生成从 1 到 `lines` 的整数序列，并遍历该序列中的每个数字 `i`。使用 `awk` 命令在第 `i` 行查找 C-α 原子，并将找到的行作为条件判断是否进入循环体。从第 `i` 行提取 C-α 原子的坐标和相关信息，并将其赋值给相应的变量。
for i in `seq 1 $lines`; do
if [ "$(awk 'NR=='$i' && ($1=="ATOM" || $1=="HETATM") && $3=="CA"{print}' $pdb)" ];then
x=`awk 'NR=='$i'{print substr($0, 31, 8)}' $pdb`
y=`awk 'NR=='$i'{print substr($0, 39, 8)}' $pdb`
z=`awk 'NR=='$i'{print substr($0, 47, 8)}' $pdb`
atom=`awk 'NR=='$i'{print $3}' $pdb`
res_num=`awk 'NR=='$i'{print substr($0, 23, 4)}' $pdb`
chain=`awk 'NR=='$i'{print substr($0, 22, 1)}' $pdb`

#输出坐标约束的各项参数，包括约束类型、原子名、残基号、链标识、约束基准的字符串 `root`，以及 C-α 坐标
echo $constraint_type $atom ${res_num}${chain} $root $x $y $z $constraint_func
fi

done
```

### 解析
用于从 PDB 文件中提取 C-α 原子的坐标输入给rosetta script进行主链限制

### 使用
```text
sh make_csts.sh example.pdb > bbCA.cst
```
### 效果


### 其它


