
# 技术背景


OpenMM是一款基于Python开发的开源分子动力学模拟软件，这几年因为AlphaFold的缘故，使得这个软件的热度有了不少提升。并且可以使用GPU硬件加速，所以性能上也不赖。这里介绍一下该软件的基本安装和使用方法，并附带一个真空蛋白体系的能量极小化示例。


# 安装OpenMM


一般这个软件首推是使用conda安装，但是这里我个人比较推荐使用python\-pip安装，原因是这样的，我先尝试了一下使用conda安装：



```
$ conda install -c conda-forge openmm
Collecting package metadata (current_repodata.json): done
Solving environment: done


==> WARNING: A newer version of conda exists. <==
  current version: 23.1.0
  latest version: 24.9.2

Please update conda by running

    $ conda update -n base -c defaults conda

Or to minimize the number of packages updated during conda update use

     conda install conda=24.9.2



## Package Plan ##

  environment location: /home/dechin/anaconda3/envs/mindspore-master

  added / updated specs:
    - openmm


The following packages will be downloaded:

    package                    |            build
    ---------------------------|-----------------
    ca-certificates-2024.8.30  |       hbcca054_0         155 KB  conda-forge
    cudatoolkit-10.2.89        |      h713d32c_10       449.7 MB  conda-forge
    libblas-3.9.0              |       8_openblas          11 KB  conda-forge
    libcblas-3.9.0             |       8_openblas          11 KB  conda-forge
    libgfortran-ng-7.5.0       |      h14aa051_20          23 KB  conda-forge
    libgfortran4-7.5.0         |      h14aa051_20         1.2 MB  conda-forge
    liblapack-3.9.0            |       8_openblas          11 KB  conda-forge
    libopenblas-0.3.12         |pthreads_hb3c22a3_1         8.2 MB  conda-forge
    numpy-1.22.3               |   py39hc58783e_2         6.8 MB  conda-forge
    ocl-icd-2.3.2              |       h5eee18b_1         136 KB  https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
    ocl-icd-system-1.0.0       |                1           4 KB  conda-forge
    openmm-8.0.0               |   py39h31b9435_1        10.1 MB  conda-forge
    python_abi-3.9             |           2_cp39           4 KB  conda-forge
    ------------------------------------------------------------
                                           Total:       476.3 MB

The following NEW packages will be INSTALLED:

  cudatoolkit        conda-forge/linux-64::cudatoolkit-10.2.89-h713d32c_10
  libblas            conda-forge/linux-64::libblas-3.9.0-8_openblas
  libcblas           conda-forge/linux-64::libcblas-3.9.0-8_openblas
  libgfortran-ng     conda-forge/linux-64::libgfortran-ng-7.5.0-h14aa051_20
  libgfortran4       conda-forge/linux-64::libgfortran4-7.5.0-h14aa051_20
  liblapack          conda-forge/linux-64::liblapack-3.9.0-8_openblas
  libopenblas        conda-forge/linux-64::libopenblas-0.3.12-pthreads_hb3c22a3_1
  numpy              conda-forge/linux-64::numpy-1.22.3-py39hc58783e_2
  ocl-icd            anaconda/pkgs/main/linux-64::ocl-icd-2.3.2-h5eee18b_1
  ocl-icd-system     conda-forge/linux-64::ocl-icd-system-1.0.0-1
  openmm             conda-forge/linux-64::openmm-8.0.0-py39h31b9435_1
  python_abi         conda-forge/linux-64::python_abi-3.9-2_cp39

The following packages will be SUPERSEDED by a higher-priority channel:

  ca-certificates    anaconda/pkgs/main::ca-certificates-2~ --> conda-forge::ca-certificates-2024.8.30-hbcca054_0


Proceed ([y]/n)? y


Downloading and Extracting Packages

Preparing transaction: done
Verifying transaction: done
Executing transaction: | By downloading and using the CUDA Toolkit conda packages, you accept the terms and conditions of the CUDA End User License Agreement (EULA): https://docs.nvidia.com/cuda/eula/index.html
                                                                                                                                                            done

```

发现这里面的配套版本比较老旧，而且也无法通过官方曾经提供的测试命令：



```
$ python3 -m simtk.testInstallation
Traceback (most recent call last):
  File "/home/dechin/anaconda3/envs/mindspore-master/lib/python3.9/runpy.py", line 188, in _run_module_as_main
    mod_name, mod_spec, code = _get_module_details(mod_name, _Error)
  File "/home/dechin/anaconda3/envs/mindspore-master/lib/python3.9/runpy.py", line 111, in _get_module_details
    __import__(pkg_name)
  File "/home/dechin/anaconda3/envs/mindspore-master/lib/python3.9/site-packages/simtk/__init__.py", line 1, in <module>
    import openmm
  File "/home/dechin/anaconda3/envs/mindspore-master/lib/python3.9/site-packages/openmm/__init__.py", line 24, in <module>
    from openmm.openmm import *
  File "/home/dechin/anaconda3/envs/mindspore-master/lib/python3.9/site-packages/openmm/openmm.py", line 10, in <module>
    from . import _openmm
ImportError: /home/dechin/anaconda3/envs/mindspore-master/lib/python3.9/site-packages/openmm/../../../libstdc++.so.6: version `GLIBCXX_3.4.30' not found (required by /home/dechin/anaconda3/envs/mindspore-master/lib/python3.9/site-packages/openmm/../../../libOpenMM.so.8.0)

```

经过一番折腾，发现这个动态链接库问题可能不是openmm导致的，而是conda环境导致的，于是切了一个conda环境，使用的python版本是：



```
$ python3 --version
Python 3.10.5

```

然后直接使用pip安装：



```
$ python3 -m pip install openmm
Looking in indexes: https://pypi.tuna.tsinghua.edu.cn/simple
Collecting openmm
  Downloading https://pypi.tuna.tsinghua.edu.cn/packages/b1/f2/db21a370b134660df2d88a704fb02ec6f7321751f700f28b4a4c16e310ff/OpenMM-8.2.0-cp310-cp310-manylinux_2_27_x86_64.manylinux_2_28_x86_64.whl (12.3 MB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 12.3/12.3 MB 5.2 MB/s eta 0:00:00
Requirement already satisfied: numpy in ./anaconda3/envs/jax/lib/python3.10/site-packages (from openmm) (1.26.4)
Installing collected packages: openmm
Successfully installed openmm-8.2.0

```

安装完成后发现这个测试指令还是不能执行：



```
$ python3 -m simtk.testInstallation
/home/dechin/anaconda3/envs/jax/bin/python3: No module named simtk.testInstallation

```

但是查看到软件版本是存在的，应该是安装成功了的：



```
$ python3 -m pip show openmm
Name: OpenMM
Version: 8.2.0
Summary: Python wrapper for OpenMM (a C++ MD package)
Home-page: https://openmm.org
Author: Peter Eastman
Author-email:
License: Python Software Foundation License (BSD-like)
Location: /home/dechin/anaconda3/envs/jax/lib/python3.10/site-packages
Requires: numpy
Required-by: pdbfixer

```

并且是最新版的`8.2.0`，于是懒得再去折腾这个testInstall了，直接进入案例测试。


# Conda安装pdbfixer


由于我们经常使用pdb格式来存储待模拟的蛋白体系，但是按照openmm的原话翻译文是，"现在很多人使用pdb的格式并未遵守其规范"，因此他们提供了一个叫`pdbfixer`的工具，用于规范化输入的pdb文件格式。安装方法如下：



```
$ conda install -c conda-forge pdbfixer
Collecting package metadata (current_repodata.json): done
Solving environment: done


==> WARNING: A newer version of conda exists. <==
  current version: 23.1.0
  latest version: 24.11.0

Please update conda by running

    $ conda update -n base -c defaults conda

Or to minimize the number of packages updated during conda update use

     conda install conda=24.11.0



## Package Plan ##

  environment location: /home/dechin/anaconda3/envs/jax

  added / updated specs:
    - pdbfixer


The following packages will be downloaded:

    package                    |            build
    ---------------------------|-----------------
    libgcc-14.2.0              |       h77fa898_1         829 KB  conda-forge
    libgcc-ng-14.2.0           |       h69a702a_1          53 KB  conda-forge
    libstdcxx-14.2.0           |       hc0a3c3a_1         3.7 MB  conda-forge
    openmm-8.2.0               |  py310h114903a_0        11.0 MB  conda-forge
    openssl-3.4.0              |       hb9d3cd8_0         2.8 MB  conda-forge
    pdbfixer-1.10              |     pyhff2d567_0         529 KB  conda-forge
    ------------------------------------------------------------
                                           Total:        18.9 MB

The following NEW packages will be INSTALLED:

  libgcc             conda-forge/linux-64::libgcc-14.2.0-h77fa898_1
  libstdcxx          conda-forge/linux-64::libstdcxx-14.2.0-hc0a3c3a_1
  pdbfixer           conda-forge/noarch::pdbfixer-1.10-pyhff2d567_0

The following packages will be UPDATED:

  libgcc-ng          anaconda/cloud/conda-forge::libgcc-ng~ --> conda-forge::libgcc-ng-14.2.0-h69a702a_1
  openmm             anaconda/cloud/conda-forge::openmm-8.~ --> conda-forge::openmm-8.2.0-py310h114903a_0
  openssl                                  3.1.3-hd590300_0 --> 3.4.0-hb9d3cd8_0


Proceed ([y]/n)? y


Downloading and Extracting Packages

Preparing transaction: done
Verifying transaction: done
Executing transaction: done

```

安装完成后可以在命令行中直接使用：



```
$ pdbfixer --help
Usage: pdbfixer
       pdbfixer filename [options]

When run with no arguments, it launches the user interface.  If any arguments are specified, it runs in command line mode.

Options:
  -h, --help            show this help message and exit
  --pdbid=PDBID         PDB id to retrieve from RCSB [default: None]
  --url=URL             URL to retrieve PDB from [default: None]
  --output=FILENAME     output pdb file [default: output.pdb]
  --add-atoms=ATOMS     which missing atoms to add: all, heavy, hydrogen, or
                        none [default: all]
  --keep-heterogens=OPTION
                        which heterogens to keep: all, water, or none
                        [default: all]
  --replace-nonstandard
                        replace nonstandard residues with standard equivalents
  --add-residues        add missing residues
  --water-box=X Y Z     add a water box. The value is the box dimensions in nm
                        [example: --water-box=2.5 2.4 3.0]
  --ph=PH               the pH to use for adding missing hydrogens [default:
                        7.0]
  --positive-ion=ION    positive ion to include in the water box: Cs+, K+,
                        Li+, Na+, or Rb+ [default: Na+]
  --negative-ion=ION    negative ion to include in the water box: Cl-, Br-,
                        F-, or I- [default: Cl-]
  --ionic-strength=STRENGTH
                        molar concentration of ions to add to the water box
                        [default: 0.0]
  --verbose             Print verbose output

```

例如检查并修复我们本地的一个`input.pdb`文件：



```
$ pdbfixer input.pdb --output=input_fix.pdb --replace-nonstandard

```

该命令行会在当前路径下生成一个`input_fix.pdb`文件。除了命令行的使用方法之外，pdbfixer还可以使用python脚本的形式来进行pdb文件修复，有相关需求的童鞋可以参考一下参考链接2中的python脚本，这里不做演示。


# 能量极小化


参考openmm官方的[cookbook文档](https://github.com)，这里我们用`amber14力场`做一个真空纯蛋白体系的能量极小化：



```
from openmm.app import *
from openmm import *
from openmm.unit import *
from sys import stdout

pdb = PDBFile('input.pdb')
forcefield = ForceField('amber14-all.xml')
system = forcefield.createSystem(pdb.topology, nonbondedCutoff=1*nanometer, constraints=HBonds)

integrator = LangevinMiddleIntegrator(300*kelvin, 1/picosecond, 0.004*picoseconds)
simulation = Simulation(pdb.topology, system, integrator)
simulation.context.setPositions(pdb.positions)

simulation.minimizeEnergy()
simulation.reporters.append(PDBReporter('output.pdb', 1000))
simulation.reporters.append(StateDataReporter(stdout, 1000, step=True,
        potentialEnergy=True, temperature=True))
simulation.step(10000)

```

这里输入的pdb文件`input.pdb`内容为：



input.pdb

```
MODEL     1
SEQRES   1 A  157  GLU GLU ALA VAL ARG MET LYS ALA GLN ASP LEU ALA LEU
SEQRES   2 A  157  ALA VAL GLN THR TYR ILE GLU ALA LYS MET LYS LEU GLU
SEQRES   3 A  157  ASN LYS THR MET LEU THR THR PHE ASP LEU ILE GLN ASP
SEQRES   4 A  157  PRO LYS PHE ARG SER LEU GLY ALA GLN ARG TRP GLY ALA
SEQRES   5 A  157  LYS GLU TYR THR TRP VAL GLY ALA GLY ASN LYS VAL ALA
SEQRES   6 A  157  GLY ARG ASP VAL ALA VAL ILE LEU THR HIS PRO ALA PHE
SEQRES   7 A  157  THR GLY GLN TYR GLU LYS TYR LEU GLY VAL ASP VAL ALA
SEQRES   8 A  157  MET LEU ARG TRP ASN GLU THR MET PRO GLU LEU TYR ASN
SEQRES   9 A  157  LEU LEU LEU LYS ILE THR GLU ASN PRO GLU ALA PRO LYS
SEQRES  10 A  157  PRO VAL CYS GLY TYR TYR HIS TRP GLU ILE PRO LYS TYR
SEQRES  11 A  157  LEU CYS HIS TYR PRO THR THR ILE LYS VAL TYR ASP PRO
SEQRES  12 A  157  ILE SER LYS GLY GLN LEU TRP VAL VAL VAL GLY THR SER
SEQRES  13 A  157  ALA
ATOM      1  N   GLU A   1       7.586  58.029  49.889   1.0   0.0           N
ATOM      2  CA  GLU A   1       8.478  58.210  48.753   1.0   0.0           C
ATOM      3  CB  GLU A   1       9.946  58.055  49.176   1.0   0.0           C
ATOM      4  CG  GLU A   1      10.499  59.224  49.986   1.0   0.0           C
ATOM      5  CD  GLU A   1      11.970  59.061  50.349   1.0   0.0           C
ATOM      6  OE1 GLU A   1      12.464  57.912  50.387   1.0   0.0           O
ATOM      7  OE2 GLU A   1      12.636  60.096  50.595   1.0   0.0           O
ATOM      8  C   GLU A   1       8.148  57.226  47.641   1.0   0.0           C
ATOM      9  O   GLU A   1       8.170  57.584  46.476   1.0   0.0           O
ATOM     10  H1  GLU A   1       7.803  58.676  50.620   1.0   0.0           H
ATOM     11  H2  GLU A   1       7.553  57.246  50.510   1.0   0.0           H
ATOM     12  H3  GLU A   1       6.606  58.227  49.923   1.0   0.0           H
ATOM     13  HA  GLU A   1       8.054  58.975  48.268   1.0   0.0           H
ATOM     14  HB2 GLU A   1       9.724  57.104  49.391   1.0   0.0           H
ATOM     15  HB3 GLU A   1       9.996  57.345  48.473   1.0   0.0           H
ATOM     16  HG2 GLU A   1      10.026  59.848  49.364   1.0   0.0           H
ATOM     17  HG3 GLU A   1       9.556  59.481  50.199   1.0   0.0           H
ATOM     18  N   GLU A   2       7.844  55.988  47.996   1.0   0.0           N
ATOM     19  CA  GLU A   2       7.430  55.014  47.006   1.0   0.0           C
ATOM     20  CB  GLU A   2       7.275  53.630  47.630   1.0   0.0           C
ATOM     21  CG  GLU A   2       7.092  52.488  46.632   1.0   0.0           C
ATOM     22  CD  GLU A   2       8.391  52.059  45.969   1.0   0.0           C
ATOM     23  OE1 GLU A   2       9.363  52.844  45.996   1.0   0.0           O
ATOM     24  OE2 GLU A   2       8.440  50.930  45.422   1.0   0.0           O
ATOM     25  C   GLU A   2       6.112  55.433  46.367   1.0   0.0           C
ATOM     26  O   GLU A   2       5.898  55.194  45.188   1.0   0.0           O
ATOM     27  H   GLU A   2       8.709  55.713  48.415   1.0   0.0           H
ATOM     28  HA  GLU A   2       7.974  55.251  46.201   1.0   0.0           H
ATOM     29  HB2 GLU A   2       8.004  53.908  48.256   1.0   0.0           H
ATOM     30  HB3 GLU A   2       7.083  54.131  48.474   1.0   0.0           H
ATOM     31  HG2 GLU A   2       6.496  52.033  47.294   1.0   0.0           H
ATOM     32  HG3 GLU A   2       6.115  52.702  46.609   1.0   0.0           H
ATOM     33  N   ALA A   3       5.226  56.054  47.133   1.0   0.0           N
ATOM     34  CA  ALA A   3       3.939  56.496  46.597   1.0   0.0           C
ATOM     35  CB  ALA A   3       3.139  57.201  47.662   1.0   0.0           C
ATOM     36  C   ALA A   3       4.104  57.417  45.407   1.0   0.0           C
ATOM     37  O   ALA A   3       3.401  57.294  44.420   1.0   0.0           O
ATOM     38  H   ALA A   3       5.757  55.586  46.426   1.0   0.0           H
ATOM     39  HA  ALA A   3       3.632  55.715  46.053   1.0   0.0           H
ATOM     40  HB1 ALA A   3       3.030  56.593  48.448   1.0   0.0           H
ATOM     41  HB2 ALA A   3       2.169  57.437  47.607   1.0   0.0           H
ATOM     42  HB3 ALA A   3       3.446  57.982  48.206   1.0   0.0           H
ATOM     43  N   VAL A   4       5.035  58.352  45.515   1.0   0.0           N
ATOM     44  CA  VAL A   4       5.348  59.240  44.412   1.0   0.0           C
ATOM     45  CB  VAL A   4       6.223  60.424  44.873   1.0   0.0           C
ATOM     46  CG1 VAL A   4       6.991  61.027  43.703   1.0   0.0           C
ATOM     47  CG2 VAL A   4       5.359  61.483  45.541   1.0   0.0           C
ATOM     48  C   VAL A   4       6.041  58.466  43.294   1.0   0.0           C
ATOM     49  O   VAL A   4       5.644  58.561  42.145   1.0   0.0           O
ATOM     50  H   VAL A   4       4.581  58.859  46.248   1.0   0.0           H
ATOM     51  HA  VAL A   4       4.482  59.257  43.912   1.0   0.0           H
ATOM     52  HB  VAL A   4       6.899  59.827  45.304   1.0   0.0           H
ATOM     53 HG11 VAL A   4       7.558  61.794  44.002   1.0   0.0           H
ATOM     54 HG12 VAL A   4       6.647  61.548  42.922   1.0   0.0           H
ATOM     55 HG13 VAL A   4       7.746  60.638  43.174   1.0   0.0           H
ATOM     56 HG21 VAL A   4       5.926  62.250  45.840   1.0   0.0           H
ATOM     57 HG22 VAL A   4       4.886  61.420  46.420   1.0   0.0           H
ATOM     58 HG23 VAL A   4       4.683  62.080  45.110   1.0   0.0           H
ATOM     59  N   ARG A   5       7.060  57.692  43.643   1.0   0.0           N
ATOM     60  CA  ARG A   5       7.829  56.956  42.652   1.0   0.0           C
ATOM     61  CB  ARG A   5       9.003  56.231  43.314   1.0   0.0           C
ATOM     62  CG  ARG A   5      10.166  55.980  42.372   1.0   0.0           C
ATOM     63  CD  ARG A   5      11.054  54.816  42.809   1.0   0.0           C
ATOM     64  NE  ARG A   5      10.302  53.585  43.107   1.0   0.0           N
ATOM     65  CZ  ARG A   5       9.672  52.808  42.212   1.0   0.0           C
ATOM     66  NH1 ARG A   5       9.674  53.092  40.909   1.0   0.0           N
ATOM     67  NH2 ARG A   5       9.018  51.727  42.624   1.0   0.0           N
ATOM     68  C   ARG A   5       6.953  55.971  41.869   1.0   0.0           C
ATOM     69  O   ARG A   5       7.156  55.768  40.682   1.0   0.0           O
ATOM     70  H   ARG A   5       6.293  58.166  43.210   1.0   0.0           H
ATOM     71  HA  ARG A   5       7.840  57.604  41.891   1.0   0.0           H
ATOM     72  HB2 ARG A   5       8.895  56.834  44.105   1.0   0.0           H
ATOM     73  HB3 ARG A   5       8.375  55.953  44.041   1.0   0.0           H
ATOM     74  HG2 ARG A   5       9.536  56.094  41.604   1.0   0.0           H
ATOM     75  HG3 ARG A   5      10.024  56.899  42.004   1.0   0.0           H
ATOM     76  HD2 ARG A   5      11.690  55.035  42.069   1.0   0.0           H
ATOM     77  HD3 ARG A   5      11.777  55.468  43.036   1.0   0.0           H
ATOM     78  HE  ARG A   5      10.257  53.302  44.065   1.0   0.0           H
ATOM     79 HH11 ARG A   5      10.166  53.905  40.599   1.0   0.0           H
ATOM     80 HH12 ARG A   5       9.205  52.513  40.242   1.0   0.0           H
ATOM     81 HH21 ARG A   5       9.017  51.514  43.601   1.0   0.0           H
ATOM     82 HH22 ARG A   5       8.549  51.148  41.957   1.0   0.0           H
ATOM     83  N   MET A   6       5.966  55.384  42.527   1.0   0.0           N
ATOM     84  CA  MET A   6       5.039  54.454  41.886   1.0   0.0           C
ATOM     85  CB  MET A   6       4.274  53.741  42.990   1.0   0.0           C
ATOM     86  CG  MET A   6       3.206  52.760  42.516   1.0   0.0           C
ATOM     87  SD  MET A   6       2.791  51.503  43.981   1.0   0.0           S
ATOM     88  CE  MET A   6       3.103  52.623  45.575   1.0   0.0           C
ATOM     89  C   MET A   6       4.071  55.177  40.993   1.0   0.0           C
ATOM     90  O   MET A   6       3.680  54.657  39.958   1.0   0.0           O
ATOM     91  H   MET A   6       6.610  54.903  43.121   1.0   0.0           H
ATOM     92  HA  MET A   6       5.528  54.026  41.126   1.0   0.0           H
ATOM     93  HB2 MET A   6       5.161  53.657  43.444   1.0   0.0           H
ATOM     94  HB3 MET A   6       4.644  54.462  43.576   1.0   0.0           H
ATOM     95  HG2 MET A   6       2.751  53.540  42.087   1.0   0.0           H
ATOM     96  HG3 MET A   6       3.351  53.145  41.605   1.0   0.0           H
ATOM     97  HE1 MET A   6       2.893  51.986  46.317   1.0   0.0           H
ATOM     98  HE2 MET A   6       2.469  53.346  45.849   1.0   0.0           H
ATOM     99  HE3 MET A   6       4.023  52.869  45.880   1.0   0.0           H
ATOM    100  N   LYS A   7       3.669  56.380  41.395   1.0   0.0           N
ATOM    101  CA  LYS A   7       2.721  57.186  40.629   1.0   0.0           C
ATOM    102  CB  LYS A   7       2.297  58.409  41.428   1.0   0.0           C
ATOM    103  CG  LYS A   7       1.146  59.189  40.820   1.0   0.0           C
ATOM    104  CD  LYS A   7      -0.184  58.524  41.129   1.0   0.0           C
ATOM    105  CE  LYS A   7      -1.269  58.937  40.148   1.0   0.0           C
ATOM    106  NZ  LYS A   7      -2.557  58.263  40.464   1.0   0.0           N
ATOM    107  C   LYS A   7       3.338  57.645  39.333   1.0   0.0           C
ATOM    108  O   LYS A   7       2.744  57.495  38.278   1.0   0.0           O
ATOM    109  H   LYS A   7       3.260  56.075  42.255   1.0   0.0           H
ATOM    110  HA  LYS A   7       2.102  56.527  40.202   1.0   0.0           H
ATOM    111  HB2 LYS A   7       2.392  57.842  42.246   1.0   0.0           H
ATOM    112  HB3 LYS A   7       3.182  58.331  41.888   1.0   0.0           H
ATOM    113  HG2 LYS A   7       1.508  60.056  41.162   1.0   0.0           H
ATOM    114  HG3 LYS A   7       1.745  59.656  40.169   1.0   0.0           H
ATOM    115  HD2 LYS A   7       0.276  57.645  41.255   1.0   0.0           H
ATOM    116  HD3 LYS A   7       0.121  58.322  42.060   1.0   0.0           H
ATOM    117  HE2 LYS A   7      -1.044  59.907  40.236   1.0   0.0           H
ATOM    118  HE3 LYS A   7      -0.615  59.230  39.451   1.0   0.0           H
ATOM    119  HZ1 LYS A   7      -3.271  58.535  39.819   1.0   0.0           H
ATOM    120  HZ2 LYS A   7      -3.152  58.410  41.254   1.0   0.0           H
ATOM    121  HZ3 LYS A   7      -2.782  57.293  40.376   1.0   0.0           H
ATOM    122  N   ALA A   8       4.530  58.227  39.419   1.0   0.0           N
ATOM    123  CA  ALA A   8       5.238  58.711  38.236   1.0   0.0           C
ATOM    124  CB  ALA A   8       6.531  59.395  38.629   1.0   0.0           C
ATOM    125  C   ALA A   8       5.523  57.588  37.263   1.0   0.0           C
ATOM    126  O   ALA A   8       5.279  57.727  36.073   1.0   0.0           O
ATOM    127  H   ALA A   8       3.676  57.775  39.160   1.0   0.0           H
ATOM    128  HA  ALA A   8       4.499  59.088  37.678   1.0   0.0           H
ATOM    129  HB1 ALA A   8       6.343  60.137  39.272   1.0   0.0           H
ATOM    130  HB2 ALA A   8       7.115  59.964  38.050   1.0   0.0           H
ATOM    131  HB3 ALA A   8       7.270  59.018  39.187   1.0   0.0           H
ATOM    132  N   GLN A   9       6.017  56.469  37.773   1.0   0.0           N
ATOM    133  CA  GLN A   9       6.328  55.335  36.924   1.0   0.0           C
ATOM    134  CB  GLN A   9       7.036  54.244  37.711   1.0   0.0           C
ATOM    135  CG  GLN A   9       7.329  52.998  36.889   1.0   0.0           C
ATOM    136  CD  GLN A   9       8.340  52.091  37.564   1.0   0.0           C
ATOM    137  OE1 GLN A   9       9.451  51.897  37.062   1.0   0.0           O
ATOM    138  NE2 GLN A   9       7.960  51.528  38.710   1.0   0.0           N
ATOM    139  C   GLN A   9       5.089  54.755  36.259   1.0   0.0           C
ATOM    140  O   GLN A   9       5.124  54.405  35.085   1.0   0.0           O
ATOM    141  H   GLN A   9       6.832  56.850  38.210   1.0   0.0           H
ATOM    142  HA  GLN A   9       6.657  55.776  36.089   1.0   0.0           H
ATOM    143  HB2 GLN A   9       7.628  54.949  38.102   1.0   0.0           H
ATOM    144  HB3 GLN A   9       6.741  54.707  38.547   1.0   0.0           H
ATOM    145  HG2 GLN A   9       6.351  52.872  36.723   1.0   0.0           H
ATOM    146  HG3 GLN A   9       6.963  53.444  36.072   1.0   0.0           H
ATOM    147 HE21 GLN A   9       7.060  51.685  39.117   1.0   0.0           H
ATOM    148 HE22 GLN A   9       8.601  50.923  39.183   1.0   0.0           H
ATOM    149  N   ASP A  10       4.005  54.638  37.012   1.0   0.0           N
ATOM    150  CA  ASP A  10       2.755  54.124  36.471   1.0   0.0           C
ATOM    151  CB  ASP A  10       1.709  53.960  37.575   1.0   0.0           C
ATOM    152  CG  ASP A  10       1.974  52.760  38.455   1.0   0.0           C
ATOM    153  OD1 ASP A  10       2.713  51.855  37.996   1.0   0.0           O
ATOM    154  OD2 ASP A  10       1.440  52.720  39.593   1.0   0.0           O
ATOM    155  C   ASP A  10       2.194  55.033  35.394   1.0   0.0           C
ATOM    156  O   ASP A  10       1.581  54.563  34.435   1.0   0.0           O
ATOM    157  H   ASP A  10       4.689  54.745  36.290   1.0   0.0           H
ATOM    158  HA  ASP A  10       3.051  53.415  35.831   1.0   0.0           H
ATOM    159  HB2 ASP A  10       1.764  54.940  37.766   1.0   0.0           H
ATOM    160  HB3 ASP A  10       1.086  54.551  37.063   1.0   0.0           H
ATOM    161  N   LEU A  11       2.371  56.336  35.570   1.0   0.0           N
ATOM    162  CA  LEU A  11       1.869  57.285  34.602   1.0   0.0           C
ATOM    163  CB  LEU A  11       1.876  58.697  35.162   1.0   0.0           C
ATOM    164  CG  LEU A  11       1.018  59.696  34.395   1.0   0.0           C
ATOM    165  CD1 LEU A  11      -0.437  59.222  34.354   1.0   0.0           C
ATOM    166  CD2 LEU A  11       1.085  61.076  35.024   1.0   0.0           C
ATOM    167  C   LEU A  11       2.721  57.218  33.363   1.0   0.0           C
ATOM    168  O   LEU A  11       2.204  57.070  32.261   1.0   0.0           O
ATOM    169  H   LEU A  11       1.805  56.381  36.393   1.0   0.0           H
ATOM    170  HA  LEU A  11       1.105  56.805  34.171   1.0   0.0           H
ATOM    171  HB2 LEU A  11       1.816  58.269  36.064   1.0   0.0           H
ATOM    172  HB3 LEU A  11       2.695  58.406  35.657   1.0   0.0           H
ATOM    173  HG  LEU A  11       1.385  59.381  33.520   1.0   0.0           H
ATOM    174 HD11 LEU A  11      -1.000  59.878  33.851   1.0   0.0           H
ATOM    175 HD12 LEU A  11      -1.090  59.169  35.109   1.0   0.0           H
ATOM    176 HD13 LEU A  11      -0.818  58.442  33.858   1.0   0.0           H
ATOM    177 HD21 LEU A  11       2.035  61.386  35.051   1.0   0.0           H
ATOM    178 HD22 LEU A  11       0.911  61.306  35.982   1.0   0.0           H
ATOM    179 HD23 LEU A  11       0.746  61.930  34.630   1.0   0.0           H
ATOM    180  N   ALA A  12       4.034  57.301  33.542   1.0   0.0           N
ATOM    181  CA  ALA A  12       4.951  57.206  32.423   1.0   0.0           C
ATOM    182  CB  ALA A  12       6.375  57.007  32.913   1.0   0.0           C
ATOM    183  C   ALA A  12       4.529  56.085  31.474   1.0   0.0           C
ATOM    184  O   ALA A  12       4.477  56.282  30.273   1.0   0.0           O
ATOM    185  H   ALA A  12       3.097  57.432  33.219   1.0   0.0           H
ATOM    186  HA  ALA A  12       4.584  57.902  31.806   1.0   0.0           H
ATOM    187  HB1 ALA A  12       6.651  57.741  33.534   1.0   0.0           H
ATOM    188  HB2 ALA A  12       7.235  57.076  32.407   1.0   0.0           H
ATOM    189  HB3 ALA A  12       6.742  56.311  33.530   1.0   0.0           H
ATOM    190  N   LEU A  13       4.210  54.918  32.018   1.0   0.0           N
ATOM    191  CA  LEU A  13       3.766  53.800  31.203   1.0   0.0           C
ATOM    192  CB  LEU A  13       3.584  52.545  32.051   1.0   0.0           C
ATOM    193  CG  LEU A  13       2.671  51.436  31.513   1.0   0.0           C
ATOM    194  CD1 LEU A  13       3.252  50.790  30.268   1.0   0.0           C
ATOM    195  CD2 LEU A  13       2.417  50.393  32.599   1.0   0.0           C
ATOM    196  C   LEU A  13       2.472  54.134  30.481   1.0   0.0           C
ATOM    197  O   LEU A  13       2.344  53.846  29.293   1.0   0.0           O
ATOM    198  H   LEU A  13       5.062  54.698  32.493   1.0   0.0           H
ATOM    199  HA  LEU A  13       4.317  53.905  30.375   1.0   0.0           H
ATOM    200  HB2 LEU A  13       4.557  52.700  32.222   1.0   0.0           H
ATOM    201  HB3 LEU A  13       3.990  53.163  32.724   1.0   0.0           H
ATOM    202  HG  LEU A  13       2.019  52.088  31.126   1.0   0.0           H
ATOM    203 HD11 LEU A  13       2.657  50.067  29.917   1.0   0.0           H
ATOM    204 HD12 LEU A  13       4.056  50.204  30.168   1.0   0.0           H
ATOM    205 HD13 LEU A  13       3.370  51.166  29.349   1.0   0.0           H
ATOM    206 HD21 LEU A  13       2.034  50.819  33.419   1.0   0.0           H
ATOM    207 HD22 LEU A  13       3.079  49.872  33.138   1.0   0.0           H
ATOM    208 HD23 LEU A  13       1.736  49.661  32.621   1.0   0.0           H
ATOM    209  N   ALA A  14       1.513  54.737  31.181   1.0   0.0           N
ATOM    210  CA  ALA A  14       0.218  55.089  30.572   1.0   0.0           C
ATOM    211  CB  ALA A  14      -0.738  55.637  31.615   1.0   0.0           C
ATOM    212  C   ALA A  14       0.364  56.096  29.429   1.0   0.0           C
ATOM    213  O   ALA A  14      -0.228  55.935  28.366   1.0   0.0           O
ATOM    214  H   ALA A  14       2.143  54.376  30.494   1.0   0.0           H
ATOM    215  HA  ALA A  14       0.057  54.315  29.959   1.0   0.0           H
ATOM    216  HB1 ALA A  14      -0.833  54.979  32.362   1.0   0.0           H
ATOM    217  HB2 ALA A  14      -1.724  55.767  31.515   1.0   0.0           H
ATOM    218  HB3 ALA A  14      -0.577  56.411  32.228   1.0   0.0           H
ATOM    219  N   VAL A  15       1.153  57.134  29.666   1.0   0.0           N
ATOM    220  CA  VAL A  15       1.454  58.110  28.638   1.0   0.0           C
ATOM    221  CB  VAL A  15       2.366  59.226  29.181   1.0   0.0           C
ATOM    222  CG1 VAL A  15       2.947  60.068  28.055   1.0   0.0           C
ATOM    223  CG2 VAL A  15       1.596  60.102  30.149   1.0   0.0           C
ATOM    224  C   VAL A  15       2.112  57.416  27.442   1.0   0.0           C
ATOM    225  O   VAL A  15       1.741  57.660  26.299   1.0   0.0           O
ATOM    226  H   VAL A  15       0.723  57.587  30.447   1.0   0.0           H
ATOM    227  HA  VAL A  15       0.581  58.195  28.158   1.0   0.0           H
ATOM    228  HB  VAL A  15       3.110  58.593  29.394   1.0   0.0           H
ATOM    229 HG11 VAL A  15       3.539  60.793  28.408   1.0   0.0           H
ATOM    230 HG12 VAL A  15       2.493  60.711  27.439   1.0   0.0           H
ATOM    231 HG13 VAL A  15       3.624  59.825  27.360   1.0   0.0           H
ATOM    232 HG21 VAL A  15       2.188  60.827  30.502   1.0   0.0           H
ATOM    233 HG22 VAL A  15       1.261  59.855  31.058   1.0   0.0           H
ATOM    234 HG23 VAL A  15       0.852  60.735  29.936   1.0   0.0           H
ATOM    235  N   GLN A  16       3.082  56.548  27.703   1.0   0.0           N
ATOM    236  CA  GLN A  16       3.790  55.873  26.625   1.0   0.0           C
ATOM    237  CB  GLN A  16       4.928  55.007  27.170   1.0   0.0           C
ATOM    238  CG  GLN A  16       5.696  54.232  26.109   1.0   0.0           C
ATOM    239  CD  GLN A  16       7.192  54.159  26.396   1.0   0.0           C
ATOM    240  OE1 GLN A  16       7.615  53.961  27.540   1.0   0.0           O
ATOM    241  NE2 GLN A  16       7.995  54.268  25.352   1.0   0.0           N
ATOM    242  C   GLN A  16       2.811  55.038  25.825   1.0   0.0           C
ATOM    243  O   GLN A  16       2.777  55.131  24.590   1.0   0.0           O
ATOM    244  H   GLN A  16       3.728  57.099  28.231   1.0   0.0           H
ATOM    245  HA  GLN A  16       3.818  56.577  25.915   1.0   0.0           H
ATOM    246  HB2 GLN A  16       5.154  55.777  27.766   1.0   0.0           H
ATOM    247  HB3 GLN A  16       4.484  55.110  28.060   1.0   0.0           H
ATOM    248  HG2 GLN A  16       4.981  53.533  26.123   1.0   0.0           H
ATOM    249  HG3 GLN A  16       4.920  54.350  25.489   1.0   0.0           H
ATOM    250 HE21 GLN A  16       7.653  54.428  24.426   1.0   0.0           H
ATOM    251 HE22 GLN A  16       8.982  54.187  25.493   1.0   0.0           H
ATOM    252  N   THR A  17       1.994  54.251  26.515   1.0   0.0           N
ATOM    253  CA  THR A  17       0.978  53.462  25.841   1.0   0.0           C
ATOM    254  CB  THR A  17       0.105  52.686  26.828   1.0   0.0           C
ATOM    255  OG1 THR A  17       0.920  51.769  27.555   1.0   0.0           O
ATOM    256  CG2 THR A  17      -0.988  51.911  26.087   1.0   0.0           C
ATOM    257  C   THR A  17       0.082  54.352  24.985   1.0   0.0           C
ATOM    258  O   THR A  17      -0.185  54.026  23.832   1.0   0.0           O
ATOM    259  H   THR A  17       2.581  53.668  27.076   1.0   0.0           H
ATOM    260  HA  THR A  17       1.473  53.073  25.064   1.0   0.0           H
ATOM    261  HB  THR A  17      -0.024  53.300  27.607   1.0   0.0           H
ATOM    262  HG1 THR A  17       0.349  51.262  28.200   1.0   0.0           H
ATOM    263 HG21 THR A  17      -1.559  51.404  26.732   1.0   0.0           H
ATOM    264 HG22 THR A  17      -0.893  51.116  25.487   1.0   0.0           H
ATOM    265 HG23 THR A  17      -1.787  52.267  25.602   1.0   0.0           H
ATOM    266  N   TYR A  18      -0.343  55.489  25.531   1.0   0.0           N
ATOM    267  CA  TYR A  18      -1.257  56.387  24.822   1.0   0.0           C
ATOM    268  CB  TYR A  18      -1.786  57.499  25.729   1.0   0.0           C
ATOM    269  CG  TYR A  18      -2.642  58.489  24.972   1.0   0.0           C
ATOM    270  CD1 TYR A  18      -3.974  58.202  24.679   1.0   0.0           C
ATOM    271  CE1 TYR A  18      -4.760  59.081  23.954   1.0   0.0           C
ATOM    272  CZ  TYR A  18      -4.208  60.267  23.507   1.0   0.0           C
ATOM    273  OH  TYR A  18      -4.990  61.144  22.779   1.0   0.0           O
ATOM    274  CE2 TYR A  18      -2.885  60.572  23.784   1.0   0.0           C
ATOM    275  CD2 TYR A  18      -2.108  59.682  24.504   1.0   0.0           C
ATOM    276  C   TYR A  18      -0.640  57.024  23.574   1.0   0.0           C
ATOM    277  O   TYR A  18      -1.288  57.088  22.535   1.0   0.0           O
ATOM    278  H   TYR A  18      -0.746  55.073  26.346   1.0   0.0           H
ATOM    279  HA  TYR A  18      -1.817  55.732  24.315   1.0   0.0           H
ATOM    280  HB2 TYR A  18      -2.039  56.826  26.424   1.0   0.0           H
ATOM    281  HB3 TYR A  18      -1.080  57.300  26.408   1.0   0.0           H
ATOM    282  HD1 TYR A  18      -4.371  57.342  25.000   1.0   0.0           H
ATOM    283  HE1 TYR A  18      -5.715  58.861  23.756   1.0   0.0           H
ATOM    284  HH  TYR A  18      -5.945  60.924  22.579   1.0   0.0           H
ATOM    285  HE2 TYR A  18      -2.492  61.434  23.464   1.0   0.0           H
ATOM    286  HD2 TYR A  18      -1.150  59.901  24.690   1.0   0.0           H
ATOM    287  N   ILE A  19       0.585  57.521  23.685   1.0   0.0           N
ATOM    288  CA  ILE A  19       1.238  58.158  22.556   1.0   0.0           C
ATOM    289  CB  ILE A  19       2.526  58.870  22.981   1.0   0.0           C
ATOM    290  CG1 ILE A  19       2.167  60.102  23.801   1.0   0.0           C
ATOM    291  CD1 ILE A  19       3.357  60.944  24.199   1.0   0.0           C
ATOM    292  CG2 ILE A  19       3.350  59.269  21.762   1.0   0.0           C
ATOM    293  C   ILE A  19       1.556  57.161  21.466   1.0   0.0           C
ATOM    294  O   ILE A  19       1.478  57.500  20.281   1.0   0.0           O
ATOM    295  H   ILE A  19       0.375  58.181  24.406   1.0   0.0           H
ATOM    296  HA  ILE A  19       0.469  58.561  22.060   1.0   0.0           H
ATOM    297  HB  ILE A  19       2.964  58.039  23.325   1.0   0.0           H
ATOM    298 HG12 ILE A  19       1.406  60.253  23.170   1.0   0.0           H
ATOM    299 HG13 ILE A  19       1.288  59.662  23.985   1.0   0.0           H
ATOM    300 HD11 ILE A  19       3.121  61.753  24.737   1.0   0.0           H
ATOM    301 HD12 ILE A  19       3.987  61.521  23.680   1.0   0.0           H
ATOM    302 HD13 ILE A  19       4.118  60.793  24.830   1.0   0.0           H
ATOM    303 HG21 ILE A  19       4.191  59.734  22.039   1.0   0.0           H
ATOM    304 HG22 ILE A  19       3.149  59.964  21.072   1.0   0.0           H
ATOM    305 HG23 ILE A  19       3.843  58.681  21.121   1.0   0.0           H
ATOM    306  N   GLU A  20       1.949  55.952  21.856   1.0   0.0           N
ATOM    307  CA  GLU A  20       2.188  54.886  20.890   1.0   0.0           C
ATOM    308  CB  GLU A  20       2.727  53.633  21.576   1.0   0.0           C
ATOM    309  CG  GLU A  20       4.211  53.684  21.911   1.0   0.0           C
ATOM    310  CD  GLU A  20       4.693  52.439  22.656   1.0   0.0           C
ATOM    311  OE1 GLU A  20       4.069  51.352  22.514   1.0   0.0           O
ATOM    312  OE2 GLU A  20       5.710  52.543  23.383   1.0   0.0           O
ATOM    313  C   GLU A  20       0.915  54.546  20.095   1.0   0.0           C
ATOM    314  O   GLU A  20       0.986  54.292  18.893   1.0   0.0           O
ATOM    315  H   GLU A  20       2.776  56.173  22.373   1.0   0.0           H
ATOM    316  HA  GLU A  20       2.594  55.389  20.127   1.0   0.0           H
ATOM    317  HB2 GLU A  20       1.891  53.642  22.124   1.0   0.0           H
ATOM    318  HB3 GLU A  20       1.896  53.184  21.247   1.0   0.0           H
ATOM    319  HG2 GLU A  20       4.393  54.019  20.986   1.0   0.0           H
ATOM    320  HG3 GLU A  20       4.191  54.672  21.761   1.0   0.0           H
ATOM    321  N   ALA A  21      -0.232  54.559  20.770   1.0   0.0           N
ATOM    322  CA  ALA A  21      -1.533  54.395  20.126   1.0   0.0           C
ATOM    323  CB  ALA A  21      -2.626  54.305  21.157   1.0   0.0           C
ATOM    324  C   ALA A  21      -1.833  55.550  19.177   1.0   0.0           C
ATOM    325  O   ALA A  21      -2.142  55.333  18.019   1.0   0.0           O
ATOM    326  H   ALA A  21       0.494  54.619  20.085   1.0   0.0           H
ATOM    327  HA  ALA A  21      -1.360  53.733  19.397   1.0   0.0           H
ATOM    328  HB1 ALA A  21      -2.429  53.547  21.779   1.0   0.0           H
ATOM    329  HB2 ALA A  21      -3.571  54.027  20.986   1.0   0.0           H
ATOM    330  HB3 ALA A  21      -2.799  54.967  21.886   1.0   0.0           H
ATOM    331  N   LYS A  22      -1.695  56.778  19.662   1.0   0.0           N
ATOM    332  CA  LYS A  22      -1.965  57.974  18.861   1.0   0.0           C
ATOM    333  CB  LYS A  22      -1.645  59.215  19.684   1.0   0.0           C
ATOM    334  CG  LYS A  22      -1.945  60.548  19.024   1.0   0.0           C
ATOM    335  CD  LYS A  22      -1.373  61.661  19.877   1.0   0.0           C
ATOM    336  CE  LYS A  22      -1.428  63.009  19.194   1.0   0.0           C
ATOM    337  NZ  LYS A  22      -2.808  63.551  19.107   1.0   0.0           N
ATOM    338  C   LYS A  22      -1.166  58.013  17.551   1.0   0.0           C
ATOM    339  O   LYS A  22      -1.713  58.321  16.502   1.0   0.0           O
ATOM    340  H   LYS A  22      -2.216  56.753  20.515   1.0   0.0           H
ATOM    341  HA  LYS A  22      -2.833  57.766  18.409   1.0   0.0           H
ATOM    342  HB2 LYS A  22      -2.023  58.693  20.448   1.0   0.0           H
ATOM    343  HB3 LYS A  22      -1.104  58.573  20.227   1.0   0.0           H
ATOM    344  HG2 LYS A  22      -1.612  60.270  18.123   1.0   0.0           H
ATOM    345  HG3 LYS A  22      -2.716  60.234  18.470   1.0   0.0           H
ATOM    346  HD2 LYS A  22      -1.870  61.301  20.666   1.0   0.0           H
ATOM    347  HD3 LYS A  22      -0.901  60.977  20.433   1.0   0.0           H
ATOM    348  HE2 LYS A  22      -0.643  63.318  19.731   1.0   0.0           H
ATOM    349  HE3 LYS A  22      -0.536  62.840  18.775   1.0   0.0           H
ATOM    350  HZ1 LYS A  22      -2.844  64.442  18.655   1.0   0.0           H
ATOM    351  HZ2 LYS A  22      -3.420  63.884  19.825   1.0   0.0           H
ATOM    352  HZ3 LYS A  22      -3.593  63.242  18.570   1.0   0.0           H
ATOM    353  N   MET A  23       0.128  57.718  17.621   1.0   0.0           N
ATOM    354  CA  MET A  23       0.987  57.750  16.440   1.0   0.0           C
ATOM    355  CB  MET A  23       2.402  57.337  16.823   1.0   0.0           C
ATOM    356  CG  MET A  23       3.129  58.530  17.419   1.0   0.0           C
ATOM    357  SD  MET A  23       4.965  58.077  17.896   1.0   0.0           S
ATOM    358  CE  MET A  23       5.005  56.098  17.898   1.0   0.0           C
ATOM    359  C   MET A  23       0.473  56.825  15.380   1.0   0.0           C
ATOM    360  O   MET A  23       0.336  57.216  14.220   1.0   0.0           O
ATOM    361  H   MET A  23       0.471  58.336  18.329   1.0   0.0           H
ATOM    362  HA  MET A  23       0.724  58.580  15.948   1.0   0.0           H
ATOM    363  HB2 MET A  23       2.109  56.516  17.313   1.0   0.0           H
ATOM    364  HB3 MET A  23       2.447  56.542  16.218   1.0   0.0           H
ATOM    365  HG2 MET A  23       2.635  59.049  16.721   1.0   0.0           H
ATOM    366  HG3 MET A  23       2.213  58.932  17.401   1.0   0.0           H
ATOM    367  HE1 MET A  23       5.946  55.866  18.143   1.0   0.0           H
ATOM    368  HE2 MET A  23       4.995  55.481  17.111   1.0   0.0           H
ATOM    369  HE3 MET A  23       4.613  55.474  18.575   1.0   0.0           H
ATOM    370  N   LYS A  24       0.208  55.579  15.759   1.0   0.0           N
ATOM    371  CA  LYS A  24      -0.255  54.563  14.812   1.0   0.0           C
ATOM    372  CB  LYS A  24      -0.035  53.153  15.382   1.0   0.0           C
ATOM    373  CG  LYS A  24       1.297  52.514  14.964   1.0   0.0           C
ATOM    374  CD  LYS A  24       2.191  52.139  16.144   1.0   0.0           C
ATOM    375  CE  LYS A  24       3.058  53.321  16.584   1.0   0.0           C
ATOM    376  NZ  LYS A  24       3.770  53.102  17.880   1.0   0.0           N
ATOM    377  C   LYS A  24      -1.712  54.783  14.417   1.0   0.0           C
ATOM    378  O   LYS A  24      -2.072  54.643  13.248   1.0   0.0           O
ATOM    379  H   LYS A  24       1.163  55.435  16.018   1.0   0.0           H
ATOM    380  HA  LYS A  24       0.056  54.964  13.950   1.0   0.0           H
ATOM    381  HB2 LYS A  24      -0.389  53.498  16.251   1.0   0.0           H
ATOM    382  HB3 LYS A  24      -1.018  53.138  15.566   1.0   0.0           H
ATOM    383  HG2 LYS A  24       0.788  51.969  14.298   1.0   0.0           H
ATOM    384  HG3 LYS A  24       1.138  52.887  14.050   1.0   0.0           H
ATOM    385  HD2 LYS A  24       1.424  51.700  16.611   1.0   0.0           H
ATOM    386  HD3 LYS A  24       2.081  51.164  15.952   1.0   0.0           H
ATOM    387  HE2 LYS A  24       3.382  53.395  15.641   1.0   0.0           H
ATOM    388  HE3 LYS A  24       2.534  53.890  15.950   1.0   0.0           H
ATOM    389  HZ1 LYS A  24       4.336  53.874  18.167   1.0   0.0           H
ATOM    390  HZ2 LYS A  24       4.519  52.494  18.143   1.0   0.0           H
ATOM    391  HZ3 LYS A  24       3.446  53.028  18.823   1.0   0.0           H
ATOM    392  N   LEU A  25      -2.530  55.182  15.377   1.0   0.0           N
ATOM    393  CA  LEU A  25      -3.944  55.482  15.130   1.0   0.0           C
ATOM    394  CB  LEU A  25      -4.660  55.654  16.477   1.0   0.0           C
ATOM    395  CG  LEU A  25      -6.131  55.282  16.678   1.0   0.0           C
ATOM    396  CD1 LEU A  25      -6.532  54.036  15.901   1.0   0.0           C
ATOM    397  CD2 LEU A  25      -6.393  55.087  18.166   1.0   0.0           C
ATOM    398  C   LEU A  25      -4.166  56.729  14.240   1.0   0.0           C
ATOM    399  O   LEU A  25      -5.250  56.904  13.694   1.0   0.0           O
ATOM    400  H   LEU A  25      -2.387  54.376  15.952   1.0   0.0           H
ATOM    401  HA  LEU A  25      -4.168  54.856  14.384   1.0   0.0           H
ATOM    402  HB2 LEU A  25      -3.725  55.394  16.718   1.0   0.0           H
ATOM    403  HB3 LEU A  25      -3.797  56.155  16.532   1.0   0.0           H
ATOM    404  HG  LEU A  25      -6.481  56.029  16.113   1.0   0.0           H
ATOM    405 HD11 LEU A  25      -7.493  53.793  16.032   1.0   0.0           H
ATOM    406 HD12 LEU A  25      -6.282  53.075  16.019   1.0   0.0           H
ATOM    407 HD13 LEU A  25      -6.634  53.873  14.920   1.0   0.0           H
ATOM    408 HD21 LEU A  25      -6.130  55.906  18.676   1.0   0.0           H
ATOM    409 HD22 LEU A  25      -5.917  54.481  18.803   1.0   0.0           H
ATOM    410 HD23 LEU A  25      -7.274  55.022  18.635   1.0   0.0           H
ATOM    411  N   GLU A  26      -3.145  57.580  14.098   1.0   0.0           N
ATOM    412  CA  GLU A  26      -3.190  58.749  13.202   1.0   0.0           C
ATOM    413  CB  GLU A  26      -2.920  60.032  13.993   1.0   0.0           C
ATOM    414  CG  GLU A  26      -4.085  60.458  14.861   1.0   0.0           C
ATOM    415  CD  GLU A  26      -3.745  61.606  15.796   1.0   0.0           C
ATOM    416  OE1 GLU A  26      -2.695  62.271  15.593   1.0   0.0           O
ATOM    417  OE2 GLU A  26      -4.539  61.835  16.738   1.0   0.0           O
ATOM    418  C   GLU A  26      -2.203  58.669  12.026   1.0   0.0           C
ATOM    419  O   GLU A  26      -2.109  59.609  11.233   1.0   0.0           O
ATOM    420  H   GLU A  26      -3.787  57.632  14.863   1.0   0.0           H
ATOM    421  HA  GLU A  26      -3.995  58.560  12.640   1.0   0.0           H
ATOM    422  HB2 GLU A  26      -2.013  59.676  14.216   1.0   0.0           H
ATOM    423  HB3 GLU A  26      -2.170  60.228  13.361   1.0   0.0           H
ATOM    424  HG2 GLU A  26      -4.678  60.403  14.057   1.0   0.0           H
ATOM    425  HG3 GLU A  26      -4.521  59.581  14.660   1.0   0.0           H
ATOM    426  N   ASN A  27      -1.455  57.575  11.930   1.0   0.0           N
ATOM    427  CA  ASN A  27      -0.540  57.362  10.827   1.0   0.0           C
ATOM    428  CB  ASN A  27      -1.321  57.305   9.507   1.0   0.0           C
ATOM    429  CG  ASN A  27      -1.079  56.017   8.749   1.0   0.0           C
ATOM    430  OD1 ASN A  27      -0.319  55.987   7.783   1.0   0.0           O
ATOM    431  ND2 ASN A  27      -1.708  54.935   9.202   1.0   0.0           N
ATOM    432  C   ASN A  27       0.590  58.401  10.772   1.0   0.0           C
ATOM    433  O   ASN A  27       1.061  58.769   9.688   1.0   0.0           O
ATOM    434  H   ASN A  27      -0.946  57.612  12.790   1.0   0.0           H
ATOM    435  HA  ASN A  27       0.078  56.694  11.241   1.0   0.0           H
ATOM    436  HB2 ASN A  27      -2.137  57.644   9.974   1.0   0.0           H
ATOM    437  HB3 ASN A  27      -1.435  58.298   9.530   1.0   0.0           H
ATOM    438 HD21 ASN A  27      -2.326  54.959   9.988   1.0   0.0           H
ATOM    439 HD22 ASN A  27      -1.556  54.062   8.738   1.0   0.0           H
ATOM    440  N   LYS A  28       1.023  58.858  11.951   1.0   0.0           N
ATOM    441  CA  LYS A  28       2.198  59.716  12.111   1.0   0.0           C
ATOM    442  CB  LYS A  28       1.882  60.915  13.016   1.0   0.0           C
ATOM    443  CG  LYS A  28       0.717  61.779  12.569   1.0   0.0           C
ATOM    444  CD  LYS A  28       0.154  62.553  13.751   1.0   0.0           C
ATOM    445  CE  LYS A  28      -0.818  63.638  13.317   1.0   0.0           C
ATOM    446  NZ  LYS A  28      -0.102  64.871  12.889   1.0   0.0           N
ATOM    447  C   LYS A  28       3.284  58.888  12.779   1.0   0.0           C
ATOM    448  O   LYS A  28       3.042  58.293  13.836   1.0   0.0           O
ATOM    449  H   LYS A  28       0.309  59.403  11.512   1.0   0.0           H
ATOM    450  HA  LYS A  28       2.655  59.769  11.223   1.0   0.0           H
ATOM    451  HB2 LYS A  28       2.044  60.287  13.777   1.0   0.0           H
ATOM    452  HB3 LYS A  28       2.798  60.755  13.384   1.0   0.0           H
ATOM    453  HG2 LYS A  28       1.219  62.141  11.783   1.0   0.0           H
ATOM    454  HG3 LYS A  28       0.548  61.248  11.738   1.0   0.0           H
ATOM    455  HD2 LYS A  28      -0.002  61.696  14.241   1.0   0.0           H
ATOM    456  HD3 LYS A  28       0.906  62.230  14.325   1.0   0.0           H
ATOM    457  HE2 LYS A  28      -1.342  62.993  12.760   1.0   0.0           H
ATOM    458  HE3 LYS A  28      -1.577  63.222  13.817   1.0   0.0           H
ATOM    459  HZ1 LYS A  28      -0.741  65.585  12.603   1.0   0.0           H
ATOM    460  HZ2 LYS A  28       0.471  65.014  12.082   1.0   0.0           H
ATOM    461  HZ3 LYS A  28       0.422  65.516  13.446   1.0   0.0           H
ATOM    462  N   THR A  29       4.482  58.861  12.208   1.0   0.0           N
ATOM    463  CA  THR A  29       5.577  58.125  12.833   1.0   0.0           C
ATOM    464  CB  THR A  29       6.500  57.492  11.790   1.0   0.0           C
ATOM    465  OG1 THR A  29       6.954  58.499  10.872   1.0   0.0           O
ATOM    466  CG2 THR A  29       5.761  56.392  11.036   1.0   0.0           C
ATOM    467  C   THR A  29       6.392  58.983  13.802   1.0   0.0           C
ATOM    468  O   THR A  29       7.246  58.456  14.523   1.0   0.0           O
ATOM    469  H   THR A  29       3.949  58.300  11.574   1.0   0.0           H
ATOM    470  HA  THR A  29       5.082  57.667  13.572   1.0   0.0           H
ATOM    471  HB  THR A  29       7.388  57.430  12.246   1.0   0.0           H
ATOM    472  HG1 THR A  29       7.557  58.085  10.190   1.0   0.0           H
ATOM    473 HG21 THR A  29       6.364  55.978  10.354   1.0   0.0           H
ATOM    474 HG22 THR A  29       5.009  56.496  10.385   1.0   0.0           H
ATOM    475 HG23 THR A  29       5.461  55.505  11.387   1.0   0.0           H
ATOM    476  N   MET A  30       6.150  60.299  13.804   1.0   0.0           N
ATOM    477  CA  MET A  30       6.660  61.190  14.870   1.0   0.0           C
ATOM    478  CB  MET A  30       8.072  61.669  14.554   1.0   0.0           C
ATOM    479  CG  MET A  30       8.732  62.358  15.740   1.0   0.0           C
ATOM    480  SD  MET A  30      10.244  63.429  14.975   1.0   0.0           S
ATOM    481  CE  MET A  30       9.431  64.785  13.781   1.0   0.0           C
ATOM    482  C   MET A  30       5.782  62.394  15.128   1.0   0.0           C
ATOM    483  O   MET A  30       5.324  63.045  14.178   1.0   0.0           O
ATOM    484  H   MET A  30       6.731  59.503  13.633   1.0   0.0           H
ATOM    485  HA  MET A  30       6.378  60.703  15.697   1.0   0.0           H
ATOM    486  HB2 MET A  30       8.265  60.793  14.111   1.0   0.0           H
ATOM    487  HB3 MET A  30       7.869  61.603  13.577   1.0   0.0           H
ATOM    488  HG2 MET A  30       7.844  62.645  16.099   1.0   0.0           H
ATOM    489  HG3 MET A  30       8.213  61.807  16.393   1.0   0.0           H
ATOM    490  HE1 MET A  30      10.185  65.319  13.399   1.0   0.0           H
ATOM    491  HE2 MET A  30       8.922  65.625  13.971   1.0   0.0           H
ATOM    492  HE3 MET A  30       9.013  64.714  12.876   1.0   0.0           H
ATOM    493  N   LEU A  31       5.572  62.689  16.427   1.0   0.0           N
ATOM    494  CA  LEU A  31       4.798  63.842  16.882   1.0   0.0           C
ATOM    495  CB  LEU A  31       3.947  63.477  18.107   1.0   0.0           C
ATOM    496  CG  LEU A  31       2.990  62.295  17.940   1.0   0.0           C
ATOM    497  CD1 LEU A  31       2.264  62.016  19.248   1.0   0.0           C
ATOM    498  CD2 LEU A  31       1.972  62.548  16.844   1.0   0.0           C
ATOM    499  C   LEU A  31       5.686  65.044  17.248   1.0   0.0           C
ATOM    500  O   LEU A  31       6.897  64.910  17.484   1.0   0.0           O
ATOM    501  H   LEU A  31       4.995  61.908  16.189   1.0   0.0           H
ATOM    502  HA  LEU A  31       4.451  64.242  16.034   1.0   0.0           H
ATOM    503  HB2 LEU A  31       4.787  63.663  18.617   1.0   0.0           H
ATOM    504  HB3 LEU A  31       4.230  64.386  18.412   1.0   0.0           H
ATOM    505  HG  LEU A  31       3.754  61.651  17.969   1.0   0.0           H
ATOM    506 HD11 LEU A  31       1.639  61.243  19.139   1.0   0.0           H
ATOM    507 HD12 LEU A  31       1.570  62.579  19.698   1.0   0.0           H
ATOM    508 HD13 LEU A  31       2.646  61.651  20.097   1.0   0.0           H
ATOM    509 HD21 LEU A  31       2.449  62.731  15.984   1.0   0.0           H
ATOM    510 HD22 LEU A  31       1.370  63.343  16.765   1.0   0.0           H
ATOM    511 HD23 LEU A  31       1.353  61.874  16.442   1.0   0.0           H
ATOM    512  N   THR A  32       5.060  66.217  17.305   1.0   0.0           N
ATOM    513  CA  THR A  32       5.690  67.439  17.819   1.0   0.0           C
ATOM    514  CB  THR A  32       5.425  68.639  16.909   1.0   0.0           C
ATOM    515  OG1 THR A  32       4.026  68.959  16.930   1.0   0.0           O
ATOM    516  CG2 THR A  32       5.847  68.322  15.491   1.0   0.0           C
ATOM    517  C   THR A  32       5.138  67.775  19.203   1.0   0.0           C
ATOM    518  O   THR A  32       4.105  67.235  19.618   1.0   0.0           O
ATOM    519  H   THR A  32       5.421  65.997  16.399   1.0   0.0           H
ATOM    520  HA  THR A  32       6.599  67.160  18.130   1.0   0.0           H
ATOM    521  HB  THR A  32       5.692  69.442  17.442   1.0   0.0           H
ATOM    522  HG1 THR A  32       3.853  69.744  16.335   1.0   0.0           H
ATOM    523 HG21 THR A  32       5.674  69.107  14.896   1.0   0.0           H
ATOM    524 HG22 THR A  32       5.419  67.677  14.858   1.0   0.0           H
ATOM    525 HG23 THR A  32       6.777  68.195  15.146   1.0   0.0           H
ATOM    526  N   THR A  33       5.826  68.675  19.909   1.0   0.0           N
ATOM    527  CA  THR A  33       5.390  69.104  21.238   1.0   0.0           C
ATOM    528  CB  THR A  33       6.220  70.304  21.775   1.0   0.0           C
ATOM    529  OG1 THR A  33       6.196  71.386  20.834   1.0   0.0           O
ATOM    530  CG2 THR A  33       7.668  69.916  22.041   1.0   0.0           C
ATOM    531  C   THR A  33       3.906  69.480  21.184   1.0   0.0           C
ATOM    532  O   THR A  33       3.117  69.090  22.043   1.0   0.0           O
ATOM    533  H   THR A  33       6.795  68.430  19.944   1.0   0.0           H
ATOM    534  HA  THR A  33       5.201  68.232  21.689   1.0   0.0           H
ATOM    535  HB  THR A  33       5.604  70.747  22.427   1.0   0.0           H
ATOM    536  HG1 THR A  33       6.730  72.158  21.179   1.0   0.0           H
ATOM    537 HG21 THR A  33       8.202  70.688  22.386   1.0   0.0           H
ATOM    538 HG22 THR A  33       8.400  69.680  21.402   1.0   0.0           H
ATOM    539 HG23 THR A  33       8.057  69.318  22.741   1.0   0.0           H
ATOM    540  N   PHE A  34       3.527  70.204  20.135   1.0   0.0           N
ATOM    541  CA  PHE A  34       2.185  70.760  20.031   1.0   0.0           C
ATOM    542  CB  PHE A  34       2.207  71.967  19.120   1.0   0.0           C
ATOM    543  CG  PHE A  34       3.143  73.032  19.597   1.0   0.0           C
ATOM    544  CD1 PHE A  34       2.855  73.761  20.726   1.0   0.0           C
ATOM    545  CE1 PHE A  34       3.720  74.737  21.175   1.0   0.0           C
ATOM    546  CZ  PHE A  34       4.894  74.992  20.491   1.0   0.0           C
ATOM    547  CE2 PHE A  34       5.194  74.269  19.357   1.0   0.0           C
ATOM    548  CD2 PHE A  34       4.323  73.287  18.919   1.0   0.0           C
ATOM    549  C   PHE A  34       1.134  69.757  19.585   1.0   0.0           C
ATOM    550  O   PHE A  34      -0.039  69.930  19.902   1.0   0.0           O
ATOM    551  H   PHE A  34       4.219  70.864  20.428   1.0   0.0           H
ATOM    552  HA  PHE A  34       1.888  70.698  20.984   1.0   0.0           H
ATOM    553  HB2 PHE A  34       2.256  71.366  18.323   1.0   0.0           H
ATOM    554  HB3 PHE A  34       1.328  71.691  18.731   1.0   0.0           H
ATOM    555  HD1 PHE A  34       2.009  73.582  21.228   1.0   0.0           H
ATOM    556  HE1 PHE A  34       3.497  75.261  21.997   1.0   0.0           H
ATOM    557  HZ  PHE A  34       5.523  75.698  20.818   1.0   0.0           H
ATOM    558  HE2 PHE A  34       6.038  74.453  18.853   1.0   0.0           H
ATOM    559  HD2 PHE A  34       4.549  72.755  18.103   1.0   0.0           H
ATOM    560  N   ASP A  35       1.543  68.707  18.877   1.0   0.0           N
ATOM    561  CA  ASP A  35       0.624  67.611  18.543   1.0   0.0           C
ATOM    562  CB  ASP A  35       1.325  66.524  17.720   1.0   0.0           C
ATOM    563  CG  ASP A  35       1.577  66.942  16.274   1.0   0.0           C
ATOM    564  OD1 ASP A  35       0.594  67.101  15.520   1.0   0.0           O
ATOM    565  OD2 ASP A  35       2.758  67.106  15.888   1.0   0.0           O
ATOM    566  C   ASP A  35       0.020  66.993  19.806   1.0   0.0           C
ATOM    567  O   ASP A  35      -1.105  66.495  19.776   1.0   0.0           O
ATOM    568  H   ASP A  35       1.086  69.416  19.414   1.0   0.0           H
ATOM    569  HA  ASP A  35      -0.216  68.102  18.310   1.0   0.0           H
ATOM    570  HB2 ASP A  35       1.946  66.368  18.488   1.0   0.0           H
ATOM    571  HB3 ASP A  35       1.054  65.857  18.414   1.0   0.0           H
ATOM    572  N   LEU A  36       0.773  67.039  20.908   1.0   0.0           N
ATOM    573  CA  LEU A  36       0.321  66.533  22.207   1.0   0.0           C
ATOM    574  CB  LEU A  36       1.503  65.973  22.987   1.0   0.0           C
ATOM    575  CG  LEU A  36       2.129  64.720  22.379   1.0   0.0           C
ATOM    576  CD1 LEU A  36       3.453  64.402  23.048   1.0   0.0           C
ATOM    577  CD2 LEU A  36       1.173  63.541  22.488   1.0   0.0           C
ATOM    578  C   LEU A  36      -0.375  67.593  23.053   1.0   0.0           C
ATOM    579  O   LEU A  36      -1.338  67.293  23.753   1.0   0.0           O
ATOM    580  H   LEU A  36       1.230  66.344  20.353   1.0   0.0           H
ATOM    581  HA  LEU A  36      -0.522  66.043  21.984   1.0   0.0           H
ATOM    582  HB2 LEU A  36       1.805  66.920  23.095   1.0   0.0           H
ATOM    583  HB3 LEU A  36       1.172  66.536  23.744   1.0   0.0           H
ATOM    584  HG  LEU A  36       2.417  65.209  21.556   1.0   0.0           H
ATOM    585 HD11 LEU A  36       3.863  63.581  22.650   1.0   0.0           H
ATOM    586 HD12 LEU A  36       3.618  64.080  23.980   1.0   0.0           H
ATOM    587 HD13 LEU A  36       4.315  64.907  22.995   1.0   0.0           H
ATOM    588 HD21 LEU A  36       0.300  63.751  22.047   1.0   0.0           H
ATOM    589 HD22 LEU A  36       0.718  63.179  23.302   1.0   0.0           H
ATOM    590 HD23 LEU A  36       1.229  62.649  22.040   1.0   0.0           H
ATOM    591  N   ILE A  37       0.101  68.832  22.984   1.0   0.0           N
ATOM    592  CA  ILE A  37      -0.559  69.948  23.673   1.0   0.0           C
ATOM    593  CB  ILE A  37       0.309  71.211  23.615   1.0   0.0           C
ATOM    594  CG1 ILE A  37       1.626  70.962  24.353   1.0   0.0           C
ATOM    595  CD1 ILE A  37       2.747  71.892  23.958   1.0   0.0           C
ATOM    596  CG2 ILE A  37      -0.425  72.399  24.217   1.0   0.0           C
ATOM    597  C   ILE A  37      -1.951  70.219  23.074   1.0   0.0           C
ATOM    598  O   ILE A  37      -2.900  70.504  23.808   1.0   0.0           O
ATOM    599  H   ILE A  37       1.005  68.656  23.373   1.0   0.0           H
ATOM    600  HA  ILE A  37      -0.951  69.494  24.473   1.0   0.0           H
ATOM    601  HB  ILE A  37       0.183  71.330  22.630   1.0   0.0           H
ATOM    602 HG12 ILE A  37       1.084  70.855  25.187   1.0   0.0           H
ATOM    603 HG13 ILE A  37       1.253  70.064  24.589   1.0   0.0           H
ATOM    604 HD11 ILE A  37       3.608  71.729  24.440   1.0   0.0           H
ATOM    605 HD12 ILE A  37       2.909  72.869  24.095   1.0   0.0           H
ATOM    606 HD13 ILE A  37       3.289  71.999  23.124   1.0   0.0           H
ATOM    607 HG21 ILE A  37       0.141  73.223  24.179   1.0   0.0           H
ATOM    608 HG22 ILE A  37      -0.668  72.565  25.173   1.0   0.0           H
ATOM    609 HG23 ILE A  37      -1.223  72.886  23.862   1.0   0.0           H
ATOM    610  N   GLN A  38      -2.054  70.111  21.746   1.0   0.0           N
ATOM    611  CA  GLN A  38      -3.341  70.146  21.046   1.0   0.0           C
ATOM    612  CB  GLN A  38      -3.162  70.058  19.513   1.0   0.0           C
ATOM    613  CG  GLN A  38      -3.477  71.338  18.739   1.0   0.0           C
ATOM    614  CD  GLN A  38      -2.444  72.439  18.946   1.0   0.0           C
ATOM    615  OE1 GLN A  38      -1.272  72.275  18.607   1.0   0.0           O
ATOM    616  NE2 GLN A  38      -2.872  73.561  19.540   1.0   0.0           N
ATOM    617  C   GLN A  38      -4.290  69.028  21.480   1.0   0.0           C
ATOM    618  O   GLN A  38      -5.500  69.208  21.445   1.0   0.0           O
ATOM    619  H   GLN A  38      -1.433  70.842  21.462   1.0   0.0           H
ATOM    620  HA  GLN A  38      -3.829  70.873  21.530   1.0   0.0           H
ATOM    621  HB2 GLN A  38      -2.333  69.539  19.723   1.0   0.0           H
ATOM    622  HB3 GLN A  38      -3.169  69.071  19.675   1.0   0.0           H
ATOM    623  HG2 GLN A  38      -3.709  70.771  17.949   1.0   0.0           H
ATOM    624  HG3 GLN A  38      -4.416  71.008  18.646   1.0   0.0           H
ATOM    625 HE21 GLN A  38      -3.824  73.694  19.815   1.0   0.0           H
ATOM    626 HE22 GLN A  38      -2.214  74.293  19.715   1.0   0.0           H
ATOM    627  N   ASP A  39      -3.750  67.868  21.840   1.0   0.0           N
ATOM    628  CA  ASP A  39      -4.565  66.705  22.180   1.0   0.0           C
ATOM    629  CB  ASP A  39      -3.695  65.442  22.153   1.0   0.0           C
ATOM    630  CG  ASP A  39      -4.508  64.158  22.233   1.0   0.0           C
ATOM    631  OD1 ASP A  39      -5.580  64.130  22.867   1.0   0.0           O
ATOM    632  OD2 ASP A  39      -4.062  63.154  21.651   1.0   0.0           O
ATOM    633  C   ASP A  39      -5.223  66.875  23.549   1.0   0.0           C
ATOM    634  O   ASP A  39      -4.550  66.790  24.566   1.0   0.0           O
ATOM    635  H   ASP A  39      -4.317  68.691  21.858   1.0   0.0           H
ATOM    636  HA  ASP A  39      -5.426  66.882  21.702   1.0   0.0           H
ATOM    637  HB2 ASP A  39      -3.160  65.848  21.412   1.0   0.0           H
ATOM    638  HB3 ASP A  39      -2.908  66.007  22.402   1.0   0.0           H
ATOM    639  N   PRO A  40      -6.549  67.090  23.585   1.0   0.0           N
ATOM    640  CA  PRO A  40      -7.237  67.290  24.870   1.0   0.0           C
ATOM    641  CB  PRO A  40      -8.675  67.597  24.456   1.0   0.0           C
ATOM    642  CG  PRO A  40      -8.842  66.905  23.142   1.0   0.0           C
ATOM    643  CD  PRO A  40      -7.501  66.964  22.462   1.0   0.0           C
ATOM    644  C   PRO A  40      -7.230  66.051  25.740   1.0   0.0           C
ATOM    645  O   PRO A  40      -7.257  66.161  26.962   1.0   0.0           O
ATOM    646  HA  PRO A  40      -6.677  67.831  25.498   1.0   0.0           H
ATOM    647  HB2 PRO A  40      -9.180  67.240  25.242   1.0   0.0           H
ATOM    648  HB3 PRO A  40      -8.923  68.476  24.863   1.0   0.0           H
ATOM    649  HG2 PRO A  40      -9.215  66.039  23.475   1.0   0.0           H
ATOM    650  HG3 PRO A  40      -9.819  67.008  22.955   1.0   0.0           H
ATOM    651  HD2 PRO A  40      -7.475  66.099  21.961   1.0   0.0           H
ATOM    652  HD3 PRO A  40      -7.587  67.396  21.564   1.0   0.0           H
ATOM    653  N   LYS A  41      -7.228  64.883  25.100   1.0   0.0           N
ATOM    654  CA  LYS A  41      -7.271  63.604  25.792   1.0   0.0           C
ATOM    655  CB  LYS A  41      -7.571  62.476  24.803   1.0   0.0           C
ATOM    656  CG  LYS A  41      -7.879  61.137  25.462   1.0   0.0           C
ATOM    657  CD  LYS A  41      -8.535  60.177  24.485   1.0   0.0           C
ATOM    658  CE  LYS A  41     -10.017  60.472  24.313   1.0   0.0           C
ATOM    659  NZ  LYS A  41     -10.476  60.115  22.944   1.0   0.0           N
ATOM    660  C   LYS A  41      -5.950  63.340  26.500   1.0   0.0           C
ATOM    661  O   LYS A  41      -5.932  62.852  27.635   1.0   0.0           O
ATOM    662  H   LYS A  41      -8.096  65.056  24.635   1.0   0.0           H
ATOM    663  HA  LYS A  41      -7.747  63.813  26.646   1.0   0.0           H
ATOM    664  HB2 LYS A  41      -8.125  63.133  24.291   1.0   0.0           H
ATOM    665  HB3 LYS A  41      -7.144  63.052  24.106   1.0   0.0           H
ATOM    666  HG2 LYS A  41      -6.968  61.102  25.872   1.0   0.0           H
ATOM    667  HG3 LYS A  41      -7.822  61.537  26.377   1.0   0.0           H
ATOM    668  HD2 LYS A  41      -7.777  60.291  23.843   1.0   0.0           H
ATOM    669  HD3 LYS A  41      -7.783  59.527  24.591   1.0   0.0           H
ATOM    670  HE2 LYS A  41     -10.252  60.028  25.177   1.0   0.0           H
ATOM    671  HE3 LYS A  41     -10.039  61.124  25.071   1.0   0.0           H
ATOM    672  HZ1 LYS A  41     -11.450  60.309  22.831   1.0   0.0           H
ATOM    673  HZ2 LYS A  41     -10.564  59.204  22.542   1.0   0.0           H
ATOM    674  HZ3 LYS A  41     -10.241  60.559  22.080   1.0   0.0           H
ATOM    675  N   PHE A  42      -4.854  63.654  25.812   1.0   0.0           N
ATOM    676  CA  PHE A  42      -3.526  63.595  26.403   1.0   0.0           C
ATOM    677  CB  PHE A  42      -2.459  63.929  25.366   1.0   0.0           C
ATOM    678  CG  PHE A  42      -1.114  64.236  25.955   1.0   0.0           C
ATOM    679  CD1 PHE A  42      -0.797  65.531  26.364   1.0   0.0           C
ATOM    680  CE1 PHE A  42       0.444  65.823  26.914   1.0   0.0           C
ATOM    681  CZ  PHE A  42       1.386  64.819  27.054   1.0   0.0           C
ATOM    682  CE2 PHE A  42       1.081  63.526  26.642   1.0   0.0           C
ATOM    683  CD2 PHE A  42      -0.162  63.240  26.099   1.0   0.0           C
ATOM    684  C   PHE A  42      -3.403  64.568  27.570   1.0   0.0           C
ATOM    685  O   PHE A  42      -2.846  64.228  28.613   1.0   0.0           O
ATOM    686  H   PHE A  42      -4.935  63.016  25.046   1.0   0.0           H
ATOM    687  HA  PHE A  42      -3.549  62.779  26.980   1.0   0.0           H
ATOM    688  HB2 PHE A  42      -2.797  63.164  24.818   1.0   0.0           H
ATOM    689  HB3 PHE A  42      -3.201  64.103  24.719   1.0   0.0           H
ATOM    690  HD1 PHE A  42      -1.473  66.260  26.260   1.0   0.0           H
ATOM    691  HE1 PHE A  42       0.656  66.755  27.208   1.0   0.0           H
ATOM    692  HZ  PHE A  42       2.282  65.022  27.449   1.0   0.0           H
ATOM    693  HE2 PHE A  42       1.761  62.799  26.738   1.0   0.0           H
ATOM    694  HD2 PHE A  42      -0.373  62.307  25.809   1.0   0.0           H
ATOM    695  N   ARG A  43      -3.899  65.784  27.387   1.0   0.0           N
ATOM    696  CA  ARG A  43      -3.766  66.802  28.415   1.0   0.0           C
ATOM    697  CB  ARG A  43      -4.225  68.146  27.865   1.0   0.0           C
ATOM    698  CG  ARG A  43      -3.299  68.670  26.776   1.0   0.0           C
ATOM    699  CD  ARG A  43      -3.255  70.180  26.736   1.0   0.0           C
ATOM    700  NE  ARG A  43      -3.042  70.690  28.071   1.0   0.0           N
ATOM    701  CZ  ARG A  43      -3.435  71.879  28.496   1.0   0.0           C
ATOM    702  NH1 ARG A  43      -4.053  72.727  27.673   1.0   0.0           N
ATOM    703  NH2 ARG A  43      -3.206  72.218  29.774   1.0   0.0           N
ATOM    704  C   ARG A  43      -4.537  66.430  29.678   1.0   0.0           C
ATOM    705  O   ARG A  43      -4.058  66.599  30.793   1.0   0.0           O
ATOM    706  H   ARG A  43      -3.598  64.902  27.748   1.0   0.0           H
ATOM    707  HA  ARG A  43      -2.896  66.555  28.842   1.0   0.0           H
ATOM    708  HB2 ARG A  43      -5.164  67.806  27.817   1.0   0.0           H
ATOM    709  HB3 ARG A  43      -4.790  68.287  28.678   1.0   0.0           H
ATOM    710  HG2 ARG A  43      -2.603  68.005  27.045   1.0   0.0           H
ATOM    711  HG3 ARG A  43      -3.338  67.785  26.313   1.0   0.0           H
ATOM    712  HD2 ARG A  43      -2.589  70.241  25.993   1.0   0.0           H
ATOM    713  HD3 ARG A  43      -3.789  70.315  25.901   1.0   0.0           H
ATOM    714  HE  ARG A  43      -2.562  70.101  28.721   1.0   0.0           H
ATOM    715 HH11 ARG A  43      -4.224  72.474  26.721   1.0   0.0           H
ATOM    716 HH12 ARG A  43      -4.350  73.626  27.994   1.0   0.0           H
ATOM    717 HH21 ARG A  43      -2.743  71.582  30.391   1.0   0.0           H
ATOM    718 HH22 ARG A  43      -3.503  73.117  30.095   1.0   0.0           H
ATOM    719  N   SER A  44      -5.739  65.908  29.485   1.0   0.0           N
ATOM    720  CA  SER A  44      -6.513  65.355  30.577   1.0   0.0           C
ATOM    721  CB  SER A  44      -7.855  64.849  30.048   1.0   0.0           C
ATOM    722  OG  SER A  44      -8.617  64.277  31.098   1.0   0.0           O
ATOM    723  C   SER A  44      -5.759  64.218  31.288   1.0   0.0           C
ATOM    724  O   SER A  44      -5.740  64.160  32.499   1.0   0.0           O
ATOM    725  H   SER A  44      -6.229  66.647  29.023   1.0   0.0           H
ATOM    726  HA  SER A  44      -6.349  65.997  31.325   1.0   0.0           H
ATOM    727  HB2 SER A  44      -8.082  65.694  29.564   1.0   0.0           H
ATOM    728  HB3 SER A  44      -7.568  64.713  29.100   1.0   0.0           H
ATOM    729  HG  SER A  44      -9.495  63.946  30.752   1.0   0.0           H
ATOM    730  N   LEU A  45      -5.153  63.319  30.519   1.0   0.0           N
ATOM    731  CA  LEU A  45      -4.440  62.146  31.052   1.0   0.0           C
ATOM    732  CB  LEU A  45      -4.015  61.213  29.911   1.0   0.0           C
ATOM    733  CG  LEU A  45      -3.208  59.960  30.280   1.0   0.0           C
ATOM    734  CD1 LEU A  45      -4.125  58.882  30.830   1.0   0.0           C
ATOM    735  CD2 LEU A  45      -2.454  59.417  29.087   1.0   0.0           C
ATOM    736  C   LEU A  45      -3.192  62.505  31.835   1.0   0.0           C
ATOM    737  O   LEU A  45      -2.941  61.944  32.902   1.0   0.0           O
ATOM    738  H   LEU A  45      -5.976  63.082  30.003   1.0   0.0           H
ATOM    739  HA  LEU A  45      -4.985  61.853  31.838   1.0   0.0           H
ATOM    740  HB2 LEU A  45      -4.928  61.438  29.571   1.0   0.0           H
ATOM    741  HB3 LEU A  45      -4.272  62.022  29.382   1.0   0.0           H
ATOM    742  HG  LEU A  45      -2.856  60.400  31.106   1.0   0.0           H
ATOM    743 HD11 LEU A  45      -3.599  58.066  31.070   1.0   0.0           H
ATOM    744 HD12 LEU A  45      -4.815  58.350  30.339   1.0   0.0           H
ATOM    745 HD13 LEU A  45      -4.619  58.877  31.699   1.0   0.0           H
ATOM    746 HD21 LEU A  45      -1.850  60.127  28.725   1.0   0.0           H
ATOM    747 HD22 LEU A  45      -2.843  59.188  28.195   1.0   0.0           H
ATOM    748 HD23 LEU A  45      -1.741  58.716  29.106   1.0   0.0           H
ATOM    749  N   GLY A  46      -2.402  63.415  31.283   1.0   0.0           N
ATOM    750  CA  GLY A  46      -1.114  63.751  31.853   1.0   0.0           C
ATOM    751  C   GLY A  46      -1.190  64.587  33.111   1.0   0.0           C
ATOM    752  O   GLY A  46      -0.400  64.396  34.027   1.0   0.0           O
ATOM    753  H   GLY A  46      -2.352  62.862  30.451   1.0   0.0           H
ATOM    754  HA2 GLY A  46      -0.839  62.807  31.673   1.0   0.0           H
ATOM    755  HA3 GLY A  46      -0.724  63.511  30.964   1.0   0.0           H
ATOM    756  N   ALA A  47      -2.139  65.517  33.155   1.0   0.0           N
ATOM    757  CA  ALA A  47      -2.335  66.384  34.319   1.0   0.0           C
ATOM    758  CB  ALA A  47      -2.626  67.804  33.877   1.0   0.0           C
ATOM    759  C   ALA A  47      -3.477  65.850  35.167   1.0   0.0           C
ATOM    760  O   ALA A  47      -4.636  65.901  34.770   1.0   0.0           O
ATOM    761  H   ALA A  47      -1.947  64.580  33.447   1.0   0.0           H
ATOM    762  HA  ALA A  47      -1.641  66.073  34.968   1.0   0.0           H
ATOM    763  HB1 ALA A  47      -1.874  68.155  33.319   1.0   0.0           H
ATOM    764  HB2 ALA A  47      -2.658  68.628  34.442   1.0   0.0           H
ATOM    765  HB3 ALA A  47      -3.320  68.115  33.228   1.0   0.0           H
ATOM    766  N   GLN A  48      -3.139  65.321  36.332   1.0   0.0           N
ATOM    767  CA  GLN A  48      -4.125  64.716  37.212   1.0   0.0           C
ATOM    768  CB  GLN A  48      -3.999  63.197  37.201   1.0   0.0           C
ATOM    769  CG  GLN A  48      -4.748  62.537  36.057   1.0   0.0           C
ATOM    770  CD  GLN A  48      -4.294  61.109  35.818   1.0   0.0           C
ATOM    771  OE1 GLN A  48      -3.341  60.624  36.452   1.0   0.0           O
ATOM    772  NE2 GLN A  48      -4.970  60.425  34.903   1.0   0.0           N
ATOM    773  C   GLN A  48      -3.959  65.206  38.628   1.0   0.0           C
ATOM    774  O   GLN A  48      -2.897  65.701  39.004   1.0   0.0           O
ATOM    775  H   GLN A  48      -3.249  64.996  35.393   1.0   0.0           H
ATOM    776  HA  GLN A  48      -4.964  65.233  37.041   1.0   0.0           H
ATOM    777  HB2 GLN A  48      -3.024  63.304  37.395   1.0   0.0           H
ATOM    778  HB3 GLN A  48      -3.707  63.250  38.156   1.0   0.0           H
ATOM    779  HG2 GLN A  48      -5.610  62.898  36.413   1.0   0.0           H
ATOM    780  HG3 GLN A  48      -5.024  63.436  35.718   1.0   0.0           H
ATOM    781 HE21 GLN A  48      -5.737  60.815  34.393   1.0   0.0           H
ATOM    782 HE22 GLN A  48      -4.701  59.480  34.716   1.0   0.0           H
ATOM    783  N   ARG A  49      -5.034  65.068  39.398   1.0   0.0           N
ATOM    784  CA  ARG A  49      -5.007  65.318  40.821   1.0   0.0           C
ATOM    785  CB  ARG A  49      -6.203  66.180  41.240   1.0   0.0           C
ATOM    786  CG  ARG A  49      -6.090  67.633  40.795   1.0   0.0           C
ATOM    787  CD  ARG A  49      -5.143  68.408  41.694   1.0   0.0           C
ATOM    788  NE  ARG A  49      -4.554  69.578  41.042   1.0   0.0           N
ATOM    789  CZ  ARG A  49      -3.659  70.386  41.617   1.0   0.0           C
ATOM    790  NH1 ARG A  49      -3.231  70.149  42.860   1.0   0.0           N
ATOM    791  NH2 ARG A  49      -3.177  71.432  40.953   1.0   0.0           N
ATOM    792  C   ARG A  49      -5.023  63.975  41.530   1.0   0.0           C
ATOM    793  O   ARG A  49      -5.655  63.029  41.079   1.0   0.0           O
ATOM    794  H   ARG A  49      -4.254  64.506  39.125   1.0   0.0           H
ATOM    795  HA  ARG A  49      -4.034  65.438  41.020   1.0   0.0           H
ATOM    796  HB2 ARG A  49      -6.803  65.431  40.958   1.0   0.0           H
ATOM    797  HB3 ARG A  49      -6.431  65.469  41.905   1.0   0.0           H
ATOM    798  HG2 ARG A  49      -5.972  67.363  39.839   1.0   0.0           H
ATOM    799  HG3 ARG A  49      -6.946  67.586  40.280   1.0   0.0           H
ATOM    800  HD2 ARG A  49      -5.803  68.344  42.443   1.0   0.0           H
ATOM    801  HD3 ARG A  49      -5.040  67.602  42.277   1.0   0.0           H
ATOM    802  HE  ARG A  49      -4.840  69.786  40.106   1.0   0.0           H
ATOM    803 HH11 ARG A  49      -3.594  69.362  43.359   1.0   0.0           H
ATOM    804 HH12 ARG A  49      -2.561  70.754  43.290   1.0   0.0           H
ATOM    805 HH21 ARG A  49      -3.497  71.609  40.022   1.0   0.0           H
ATOM    806 HH22 ARG A  49      -2.507  72.037  41.383   1.0   0.0           H
ATOM    807  N   TRP A  50      -4.295  63.891  42.630   1.0   0.0           N
ATOM    808  CA  TRP A  50      -4.341  62.708  43.467   1.0   0.0           C
ATOM    809  CB  TRP A  50      -3.425  61.613  42.925   1.0   0.0           C
ATOM    810  CG  TRP A  50      -1.954  61.948  43.006   1.0   0.0           C
ATOM    811  CD1 TRP A  50      -1.272  62.799  42.198   1.0   0.0           C
ATOM    812  NE1 TRP A  50       0.054  62.853  42.571   1.0   0.0           N
ATOM    813  CE2 TRP A  50       0.247  62.022  43.638   1.0   0.0           C
ATOM    814  CD2 TRP A  50      -0.998  61.433  43.940   1.0   0.0           C
ATOM    815  CE3 TRP A  50      -1.074  60.534  44.998   1.0   0.0           C
ATOM    816  CZ3 TRP A  50       0.075  60.249  45.707   1.0   0.0           C
ATOM    817  CH2 TRP A  50       1.302  60.851  45.387   1.0   0.0           C
ATOM    818  CZ2 TRP A  50       1.405  61.742  44.358   1.0   0.0           C
ATOM    819  C   TRP A  50      -3.960  63.050  44.894   1.0   0.0           C
ATOM    820  O   TRP A  50      -3.314  64.069  45.148   1.0   0.0           O
ATOM    821  H   TRP A  50      -4.546  63.665  41.689   1.0   0.0           H
ATOM    822  HA  TRP A  50      -5.314  62.642  43.690   1.0   0.0           H
ATOM    823  HB2 TRP A  50      -3.998  60.948  43.404   1.0   0.0           H
ATOM    824  HB3 TRP A  50      -4.227  61.240  42.458   1.0   0.0           H
ATOM    825  HD1 TRP A  50      -1.674  63.313  41.440   1.0   0.0           H
ATOM    826  HE1 TRP A  50       0.759  63.410  42.132   1.0   0.0           H
ATOM    827  HE3 TRP A  50      -1.943  60.103  45.240   1.0   0.0           H
ATOM    828  HZ3 TRP A  50       0.033  59.599  46.466   1.0   0.0           H
ATOM    829  HH2 TRP A  50       2.115  60.623  45.922   1.0   0.0           H
ATOM    830  HZ2 TRP A  50       2.277  62.176  44.131   1.0   0.0           H
ATOM    831  N   GLY A  51      -4.400  62.206  45.819   1.0   0.0           N
ATOM    832  CA  GLY A  51      -3.975  62.295  47.204   1.0   0.0           C
ATOM    833  C   GLY A  51      -4.260  63.632  47.862   1.0   0.0           C
ATOM    834  O   GLY A  51      -5.397  64.104  47.866   1.0   0.0           O
ATOM    835  H   GLY A  51      -4.212  61.325  45.385   1.0   0.0           H
ATOM    836  HA2 GLY A  51      -4.346  61.374  47.324   1.0   0.0           H
ATOM    837  HA3 GLY A  51      -3.405  61.496  47.012   1.0   0.0           H
ATOM    838  N   ALA A  52      -3.204  64.243  48.401   1.0   0.0           N
ATOM    839  CA  ALA A  52      -3.295  65.503  49.151   1.0   0.0           C
ATOM    840  CB  ALA A  52      -2.067  65.646  50.047   1.0   0.0           C
ATOM    841  C   ALA A  52      -3.417  66.725  48.232   1.0   0.0           C
ATOM    842  O   ALA A  52      -2.683  67.701  48.385   1.0   0.0           O
ATOM    843  H   ALA A  52      -4.008  64.149  47.814   1.0   0.0           H
ATOM    844  HA  ALA A  52      -4.256  65.542  49.424   1.0   0.0           H
ATOM    845  HB1 ALA A  52      -1.987  64.849  50.646   1.0   0.0           H
ATOM    846  HB2 ALA A  52      -1.932  66.294  50.797   1.0   0.0           H
ATOM    847  HB3 ALA A  52      -1.106  65.607  49.774   1.0   0.0           H
ATOM    848  N   LYS A  53      -4.350  66.671  47.290   1.0   0.0           N
ATOM    849  CA  LYS A  53      -4.388  67.637  46.207   1.0   0.0           C
ATOM    850  CB  LYS A  53      -4.978  68.964  46.680   1.0   0.0           C
ATOM    851  CG  LYS A  53      -6.495  68.954  46.690   1.0   0.0           C
ATOM    852  CD  LYS A  53      -7.069  70.329  46.985   1.0   0.0           C
ATOM    853  CE  LYS A  53      -8.500  70.450  46.478   1.0   0.0           C
ATOM    854  NZ  LYS A  53      -9.301  71.388  47.315   1.0   0.0           N
ATOM    855  C   LYS A  53      -3.000  67.826  45.597   1.0   0.0           C
ATOM    856  O   LYS A  53      -2.585  68.942  45.281   1.0   0.0           O
ATOM    857  H   LYS A  53      -5.258  66.547  47.689   1.0   0.0           H
ATOM    858  HA  LYS A  53      -4.661  67.048  45.446   1.0   0.0           H
ATOM    859  HB2 LYS A  53      -4.317  68.987  47.430   1.0   0.0           H
ATOM    860  HB3 LYS A  53      -4.105  69.414  46.492   1.0   0.0           H
ATOM    861  HG2 LYS A  53      -6.508  68.416  45.847   1.0   0.0           H
ATOM    862  HG3 LYS A  53      -6.502  67.962  46.813   1.0   0.0           H
ATOM    863  HD2 LYS A  53      -6.729  70.297  47.925   1.0   0.0           H
ATOM    864  HD3 LYS A  53      -6.157  70.721  47.108   1.0   0.0           H
ATOM    865  HE2 LYS A  53      -8.188  70.559  45.534   1.0   0.0           H
ATOM    866  HE3 LYS A  53      -8.440  69.584  45.981   1.0   0.0           H
ATOM    867  HZ1 LYS A  53     -10.241  71.467  46.982   1.0   0.0           H
ATOM    868  HZ2 LYS A  53      -9.217  72.381  47.404   1.0   0.0           H
ATOM    869  HZ3 LYS A  53      -9.613  71.279  48.259   1.0   0.0           H
ATOM    870  N   GLU A  54      -2.285  66.716  45.438   1.0   0.0           N
ATOM    871  CA  GLU A  54      -1.048  66.703  44.677   1.0   0.0           C
ATOM    872  CB  GLU A  54      -0.112  65.596  45.145   1.0   0.0           C
ATOM    873  CG  GLU A  54       0.312  65.737  46.590   1.0   0.0           C
ATOM    874  CD  GLU A  54       1.208  66.935  46.832   1.0   0.0           C
ATOM    875  OE1 GLU A  54       1.699  67.550  45.857   1.0   0.0           O
ATOM    876  OE2 GLU A  54       1.444  67.232  48.018   1.0   0.0           O
ATOM    877  C   GLU A  54      -1.448  66.471  43.247   1.0   0.0           C
ATOM    878  O   GLU A  54      -2.612  66.202  42.961   1.0   0.0           O
ATOM    879  H   GLU A  54      -2.019  66.870  46.389   1.0   0.0           H
ATOM    880  HA  GLU A  54      -0.825  67.664  44.515   1.0   0.0           H
ATOM    881  HB2 GLU A  54      -0.701  64.936  44.679   1.0   0.0           H
ATOM    882  HB3 GLU A  54       0.076  65.394  44.184   1.0   0.0           H
ATOM    883  HG2 GLU A  54      -0.629  65.542  46.867   1.0   0.0           H
ATOM    884  HG3 GLU A  54       0.061  64.780  46.734   1.0   0.0           H
ATOM    885  N   TYR A  55      -0.480  66.548  42.348   1.0   0.0           N
ATOM    886  CA  TYR A  55      -0.806  66.696  40.951   1.0   0.0           C
ATOM    887  CB  TYR A  55      -1.243  68.153  40.647   1.0   0.0           C
ATOM    888  CG  TYR A  55      -0.208  69.283  40.834   1.0   0.0           C
ATOM    889  CD1 TYR A  55       0.757  69.260  41.860   1.0   0.0           C
ATOM    890  CE1 TYR A  55       1.674  70.300  42.007   1.0   0.0           C
ATOM    891  CZ  TYR A  55       1.624  71.393  41.134   1.0   0.0           C
ATOM    892  OH  TYR A  55       2.515  72.434  41.258   1.0   0.0           O
ATOM    893  CE2 TYR A  55       0.672  71.449  40.137   1.0   0.0           C
ATOM    894  CD2 TYR A  55      -0.233  70.408  39.994   1.0   0.0           C
ATOM    895  C   TYR A  55       0.319  66.193  40.053   1.0   0.0           C
ATOM    896  O   TYR A  55       1.502  66.373  40.351   1.0   0.0           O
ATOM    897  H   TYR A  55      -1.218  66.878  42.937   1.0   0.0           H
ATOM    898  HA  TYR A  55      -1.242  65.805  40.826   1.0   0.0           H
ATOM    899  HB2 TYR A  55      -1.729  67.684  39.909   1.0   0.0           H
ATOM    900  HB3 TYR A  55      -2.128  67.688  40.685   1.0   0.0           H
ATOM    901  HD1 TYR A  55       0.785  68.486  42.493   1.0   0.0           H
ATOM    902  HE1 TYR A  55       2.363  70.265  42.731   1.0   0.0           H
ATOM    903  HH  TYR A  55       3.205  72.393  41.981   1.0   0.0           H
ATOM    904  HE2 TYR A  55       0.635  72.236  39.521   1.0   0.0           H
ATOM    905  HD2 TYR A  55      -0.923  70.460  39.272   1.0   0.0           H
ATOM    906  N   THR A  56      -0.079  65.507  38.984   1.0   0.0           N
ATOM    907  CA  THR A  56       0.858  64.948  38.018   1.0   0.0           C
ATOM    908  CB  THR A  56       0.389  63.596  37.443   1.0   0.0           C
ATOM    909  OG1 THR A  56      -0.728  63.797  36.578   1.0   0.0           O
ATOM    910  CG2 THR A  56      -0.013  62.630  38.544   1.0   0.0           C
ATOM    911  C   THR A  56       1.013  65.899  36.867   1.0   0.0           C
ATOM    912  O   THR A  56       0.242  66.842  36.703   1.0   0.0           O
ATOM    913  H   THR A  56      -0.182  64.873  39.751   1.0   0.0           H
ATOM    914  HA  THR A  56       1.763  65.107  38.412   1.0   0.0           H
ATOM    915  HB  THR A  56       1.036  63.401  36.705   1.0   0.0           H
ATOM    916  HG1 THR A  56      -1.032  62.920  36.205   1.0   0.0           H
ATOM    917 HG21 THR A  56      -0.317  61.753  38.171   1.0   0.0           H
ATOM    918 HG22 THR A  56      -0.796  62.659  39.166   1.0   0.0           H
ATOM    919 HG23 THR A  56       0.553  62.165  39.225   1.0   0.0           H
ATOM    920  N   TRP A  57       2.017  65.639  36.057   1.0   0.0           N
ATOM    921  CA  TRP A  57       2.250  66.466  34.896   1.0   0.0           C
ATOM    922  CB  TRP A  57       2.909  67.772  35.313   1.0   0.0           C
ATOM    923  CG  TRP A  57       4.232  67.572  35.950   1.0   0.0           C
ATOM    924  CD1 TRP A  57       5.419  67.387  35.312   1.0   0.0           C
ATOM    925  NE1 TRP A  57       6.424  67.231  36.230   1.0   0.0           N
ATOM    926  CE2 TRP A  57       5.897  67.313  37.491   1.0   0.0           C
ATOM    927  CD2 TRP A  57       4.514  67.522  37.353   1.0   0.0           C
ATOM    928  CE3 TRP A  57       3.731  67.642  38.505   1.0   0.0           C
ATOM    929  CZ3 TRP A  57       4.344  67.545  39.739   1.0   0.0           C
ATOM    930  CH2 TRP A  57       5.721  67.339  39.843   1.0   0.0           C
ATOM    931  CZ2 TRP A  57       6.514  67.219  38.732   1.0   0.0           C
ATOM    932  C   TRP A  57       3.126  65.741  33.883   1.0   0.0           C
ATOM    933  O   TRP A  57       3.668  64.664  34.159   1.0   0.0           O
ATOM    934  H   TRP A  57       1.442  66.115  36.722   1.0   0.0           H
ATOM    935  HA  TRP A  57       1.422  66.350  34.348   1.0   0.0           H
ATOM    936  HB2 TRP A  57       2.666  68.177  34.432   1.0   0.0           H
ATOM    937  HB3 TRP A  57       2.021  68.231  35.286   1.0   0.0           H
ATOM    938  HD1 TRP A  57       5.540  67.368  34.320   1.0   0.0           H
ATOM    939  HE1 TRP A  57       7.389  67.081  36.014   1.0   0.0           H
ATOM    940  HE3 TRP A  57       2.745  67.795  38.437   1.0   0.0           H
ATOM    941  HZ3 TRP A  57       3.793  67.623  40.570   1.0   0.0           H
ATOM    942  HH2 TRP A  57       6.138  67.278  40.750   1.0   0.0           H
ATOM    943  HZ2 TRP A  57       7.500  67.069  38.812   1.0   0.0           H
ATOM    944  N   VAL A  58       3.250  66.342  32.708   1.0   0.0           N
ATOM    945  CA  VAL A  58       4.063  65.781  31.648   1.0   0.0           C
ATOM    946  CB  VAL A  58       3.197  65.067  30.583   1.0   0.0           C
ATOM    947  CG1 VAL A  58       4.050  64.191  29.676   1.0   0.0           C
ATOM    948  CG2 VAL A  58       2.131  64.221  31.248   1.0   0.0           C
ATOM    949  C   VAL A  58       4.862  66.901  31.014   1.0   0.0           C
ATOM    950  O   VAL A  58       4.365  68.010  30.845   1.0   0.0           O
ATOM    951  H   VAL A  58       2.723  65.603  33.127   1.0   0.0           H
ATOM    952  HA  VAL A  58       4.864  65.436  32.138   1.0   0.0           H
ATOM    953  HB  VAL A  58       3.075  65.905  30.051   1.0   0.0           H
ATOM    954 HG11 VAL A  58       3.490  63.730  28.988   1.0   0.0           H
ATOM    955 HG12 VAL A  58       4.536  63.343  29.888   1.0   0.0           H
ATOM    956 HG13 VAL A  58       4.724  64.459  28.987   1.0   0.0           H
ATOM    957 HG21 VAL A  58       1.571  63.760  30.560   1.0   0.0           H
ATOM    958 HG22 VAL A  58       1.330  64.531  31.761   1.0   0.0           H
ATOM    959 HG23 VAL A  58       2.253  63.383  31.780   1.0   0.0           H
ATOM    960  N   GLY A  59       6.107  66.598  30.685   1.0   0.0           N
ATOM    961  CA  GLY A  59       6.965  67.525  29.989   1.0   0.0           C
ATOM    962  C   GLY A  59       7.830  66.774  28.999   1.0   0.0           C
ATOM    963  O   GLY A  59       7.694  65.570  28.837   1.0   0.0           O
ATOM    964  H   GLY A  59       5.536  67.094  31.339   1.0   0.0           H
ATOM    965  HA2 GLY A  59       6.221  68.158  29.775   1.0   0.0           H
ATOM    966  HA3 GLY A  59       6.906  68.245  30.680   1.0   0.0           H
ATOM    967  N   ALA A  60       8.732  67.485  28.340   1.0   0.0           N
ATOM    968  CA  ALA A  60       9.664  66.859  27.425   1.0   0.0           C
ATOM    969  CB  ALA A  60       9.145  66.951  26.012   1.0   0.0           C
ATOM    970  C   ALA A  60      11.001  67.547  27.542   1.0   0.0           C
ATOM    971  O   ALA A  60      11.057  68.747  27.775   1.0   0.0           O
ATOM    972  H   ALA A  60       9.076  67.424  29.277   1.0   0.0           H
ATOM    973  HA  ALA A  60       9.985  66.045  27.909   1.0   0.0           H
ATOM    974  HB1 ALA A  60       8.259  66.495  25.934   1.0   0.0           H
ATOM    975  HB2 ALA A  60       9.505  66.500  25.195   1.0   0.0           H
ATOM    976  HB3 ALA A  60       8.824  67.765  25.528   1.0   0.0           H
ATOM    977  N   GLY A  61      12.081  66.796  27.385   1.0   0.0           N
ATOM    978  CA  GLY A  61      13.389  67.381  27.537   1.0   0.0           C
ATOM    979  C   GLY A  61      14.538  66.535  27.055   1.0   0.0           C
ATOM    980  O   GLY A  61      14.467  65.317  27.013   1.0   0.0           O
ATOM    981  H   GLY A  61      11.318  67.358  27.705   1.0   0.0           H
ATOM    982  HA2 GLY A  61      12.963  68.232  27.229   1.0   0.0           H
ATOM    983  HA3 GLY A  61      12.934  68.000  28.178   1.0   0.0           H
ATOM    984  N   ASN A  62      15.600  67.225  26.666   1.0   0.0           N
ATOM    985  CA  ASN A  62      16.829  66.603  26.234   1.0   0.0           C
ATOM    986  CB  ASN A  62      16.623  65.876  24.907   1.0   0.0           C
ATOM    987  CG  ASN A  62      17.670  64.823  24.661   1.0   0.0           C
ATOM    988  OD1 ASN A  62      18.568  64.614  25.488   1.0   0.0           O
ATOM    989  ND2 ASN A  62      17.565  64.139  23.525   1.0   0.0           N
ATOM    990  C   ASN A  62      17.884  67.682  26.057   1.0   0.0           C
ATOM    991  O   ASN A  62      17.565  68.871  26.118   1.0   0.0           O
ATOM    992  H   ASN A  62      15.735  67.701  27.535   1.0   0.0           H
ATOM    993  HA  ASN A  62      17.249  66.228  27.060   1.0   0.0           H
ATOM    994  HB2 ASN A  62      15.646  65.789  25.102   1.0   0.0           H
ATOM    995  HB3 ASN A  62      16.047  66.625  24.580   1.0   0.0           H
ATOM    996 HD21 ASN A  62      16.840  64.308  22.857   1.0   0.0           H
ATOM    997 HD22 ASN A  62      18.239  63.426  23.331   1.0   0.0           H
ATOM    998  N   LYS A  63      19.136  67.276  25.859   1.0   0.0           N
ATOM    999  CA  LYS A  63      20.163  68.205  25.407   1.0   0.0           C
ATOM   1000  CB  LYS A  63      21.556  67.745  25.815   1.0   0.0           C
ATOM   1001  CG  LYS A  63      21.799  67.867  27.303   1.0   0.0           C
ATOM   1002  CD  LYS A  63      23.263  67.659  27.649   1.0   0.0           C
ATOM   1003  CE  LYS A  63      23.416  67.140  29.074   1.0   0.0           C
ATOM   1004  NZ  LYS A  63      24.832  66.841  29.420   1.0   0.0           N
ATOM   1005  C   LYS A  63      20.077  68.338  23.897   1.0   0.0           C
ATOM   1006  O   LYS A  63      19.960  67.339  23.190   1.0   0.0           O
ATOM   1007  H   LYS A  63      19.193  67.188  26.854   1.0   0.0           H
ATOM   1008  HA  LYS A  63      19.743  69.099  25.562   1.0   0.0           H
ATOM   1009  HB2 LYS A  63      21.455  66.942  25.228   1.0   0.0           H
ATOM   1010  HB3 LYS A  63      21.856  67.842  24.866   1.0   0.0           H
ATOM   1011  HG2 LYS A  63      21.244  68.698  27.345   1.0   0.0           H
ATOM   1012  HG3 LYS A  63      20.830  67.715  27.499   1.0   0.0           H
ATOM   1013  HD2 LYS A  63      23.435  67.184  26.786   1.0   0.0           H
ATOM   1014  HD3 LYS A  63      23.571  68.261  26.913   1.0   0.0           H
ATOM   1015  HE2 LYS A  63      22.808  67.859  29.412   1.0   0.0           H
ATOM   1016  HE3 LYS A  63      22.448  66.890  29.101   1.0   0.0           H
ATOM   1017  HZ1 LYS A  63      24.932  66.501  30.355   1.0   0.0           H
ATOM   1018  HZ2 LYS A  63      25.607  67.459  29.551   1.0   0.0           H
ATOM   1019  HZ3 LYS A  63      25.440  66.122  29.082   1.0   0.0           H
ATOM   1020  N   VAL A  64      20.102  69.581  23.422   1.0   0.0           N
ATOM   1021  CA  VAL A  64      20.119  69.891  22.002   1.0   0.0           C
ATOM   1022  CB  VAL A  64      18.728  70.317  21.497   1.0   0.0           C
ATOM   1023  CG1 VAL A  64      18.787  70.708  20.025   1.0   0.0           C
ATOM   1024  CG2 VAL A  64      17.714  69.203  21.702   1.0   0.0           C
ATOM   1025  C   VAL A  64      21.113  71.029  21.813   1.0   0.0           C
ATOM   1026  O   VAL A  64      21.065  72.028  22.534   1.0   0.0           O
ATOM   1027  H   VAL A  64      19.449  68.834  23.546   1.0   0.0           H
ATOM   1028  HA  VAL A  64      20.740  69.217  21.602   1.0   0.0           H
ATOM   1029  HB  VAL A  64      18.789  71.200  21.962   1.0   0.0           H
ATOM   1030 HG11 VAL A  64      17.884  70.985  19.697   1.0   0.0           H
ATOM   1031 HG12 VAL A  64      18.940  70.121  19.230   1.0   0.0           H
ATOM   1032 HG13 VAL A  64      19.226  71.504  19.609   1.0   0.0           H
ATOM   1033 HG21 VAL A  64      16.811  69.480  21.374   1.0   0.0           H
ATOM   1034 HG22 VAL A  64      17.336  68.851  22.558   1.0   0.0           H
ATOM   1035 HG23 VAL A  64      17.653  68.320  21.237   1.0   0.0           H
ATOM   1036  N   ALA A  65      22.010  70.872  20.841   1.0   0.0           N
ATOM   1037  CA  ALA A  65      23.173  71.755  20.691   1.0   0.0           C
ATOM   1038  CB  ALA A  65      22.738  73.162  20.286   1.0   0.0           C
ATOM   1039  C   ALA A  65      24.000  71.779  21.989   1.0   0.0           C
ATOM   1040  O   ALA A  65      24.502  72.824  22.406   1.0   0.0           O
ATOM   1041  H   ALA A  65      22.295  69.951  21.106   1.0   0.0           H
ATOM   1042  HA  ALA A  65      23.832  71.142  20.254   1.0   0.0           H
ATOM   1043  HB1 ALA A  65      22.201  73.146  19.443   1.0   0.0           H
ATOM   1044  HB2 ALA A  65      23.289  73.926  19.949   1.0   0.0           H
ATOM   1045  HB3 ALA A  65      22.079  73.775  20.723   1.0   0.0           H
ATOM   1046  N   GLY A  66      24.131  70.608  22.613   1.0   0.0           N
ATOM   1047  CA  GLY A  66      24.828  70.469  23.886   1.0   0.0           C
ATOM   1048  C   GLY A  66      24.294  71.378  24.979   1.0   0.0           C
ATOM   1049  O   GLY A  66      25.050  71.830  25.841   1.0   0.0           O
ATOM   1050  H   GLY A  66      24.483  70.009  21.893   1.0   0.0           H
ATOM   1051  HA2 GLY A  66      24.801  69.478  23.752   1.0   0.0           H
ATOM   1052  HA3 GLY A  66      25.568  70.027  23.379   1.0   0.0           H
ATOM   1053  N   ARG A  67      22.987  71.629  24.953   1.0   0.0           N
ATOM   1054  CA  ARG A  67      22.361  72.587  25.856   1.0   0.0           C
ATOM   1055  CB  ARG A  67      22.197  73.930  25.142   1.0   0.0           C
ATOM   1056  CG  ARG A  67      21.743  75.086  26.028   1.0   0.0           C
ATOM   1057  CD  ARG A  67      21.569  76.368  25.221   1.0   0.0           C
ATOM   1058  NE  ARG A  67      20.929  76.119  23.923   1.0   0.0           N
ATOM   1059  CZ  ARG A  67      19.630  75.868  23.738   1.0   0.0           C
ATOM   1060  NH1 ARG A  67      18.785  75.825  24.769   1.0   0.0           N
ATOM   1061  NH2 ARG A  67      19.166  75.653  22.506   1.0   0.0           N
ATOM   1062  C   ARG A  67      21.007  72.052  26.293   1.0   0.0           C
ATOM   1063  O   ARG A  67      20.197  71.660  25.454   1.0   0.0           O
ATOM   1064  H   ARG A  67      23.094  70.751  25.420   1.0   0.0           H
ATOM   1065  HA  ARG A  67      22.771  72.407  26.750   1.0   0.0           H
ATOM   1066  HB2 ARG A  67      23.045  73.696  24.666   1.0   0.0           H
ATOM   1067  HB3 ARG A  67      22.202  73.379  24.308   1.0   0.0           H
ATOM   1068  HG2 ARG A  67      21.052  74.510  26.465   1.0   0.0           H
ATOM   1069  HG3 ARG A  67      22.054  74.616  26.854   1.0   0.0           H
ATOM   1070  HD2 ARG A  67      21.208  76.853  26.017   1.0   0.0           H
ATOM   1071  HD3 ARG A  67      22.282  76.837  25.742   1.0   0.0           H
ATOM   1072  HE  ARG A  67      21.513  76.139  23.112   1.0   0.0           H
ATOM   1073 HH11 ARG A  67      19.133  75.986  25.693   1.0   0.0           H
ATOM   1074 HH12 ARG A  67      17.813  75.637  24.631   1.0   0.0           H
ATOM   1075 HH21 ARG A  67      19.800  75.685  21.733   1.0   0.0           H
ATOM   1076 HH22 ARG A  67      18.194  75.465  22.368   1.0   0.0           H
ATOM   1077  N   ASP A  68      20.755  72.060  27.601   1.0   0.0           N
ATOM   1078  CA  ASP A  68      19.525  71.487  28.143   1.0   0.0           C
ATOM   1079  CB  ASP A  68      19.573  71.440  29.674   1.0   0.0           C
ATOM   1080  CG  ASP A  68      20.528  70.370  30.201   1.0   0.0           C
ATOM   1081  OD1 ASP A  68      20.267  69.165  29.988   1.0   0.0           O
ATOM   1082  OD2 ASP A  68      21.532  70.733  30.846   1.0   0.0           O
ATOM   1083  C   ASP A  68      18.278  72.245  27.667   1.0   0.0           C
ATOM   1084  O   ASP A  68      18.263  73.473  27.607   1.0   0.0           O
ATOM   1085  H   ASP A  68      20.724  72.091  26.602   1.0   0.0           H
ATOM   1086  HA  ASP A  68      19.363  70.738  27.501   1.0   0.0           H
ATOM   1087  HB2 ASP A  68      19.621  72.439  29.691   1.0   0.0           H
ATOM   1088  HB3 ASP A  68      18.709  71.943  29.701   1.0   0.0           H
ATOM   1089  N   VAL A  69      17.249  71.483  27.301   1.0   0.0           N
ATOM   1090  CA  VAL A  69      15.977  72.023  26.822   1.0   0.0           C
ATOM   1091  CB  VAL A  69      15.820  71.847  25.292   1.0   0.0           C
ATOM   1092  CG1 VAL A  69      14.451  72.318  24.815   1.0   0.0           C
ATOM   1093  CG2 VAL A  69      16.919  72.596  24.554   1.0   0.0           C
ATOM   1094  C   VAL A  69      14.873  71.249  27.522   1.0   0.0           C
ATOM   1095  O   VAL A  69      14.871  70.027  27.493   1.0   0.0           O
ATOM   1096  H   VAL A  69      17.976  71.992  26.840   1.0   0.0           H
ATOM   1097  HA  VAL A  69      15.820  72.862  27.344   1.0   0.0           H
ATOM   1098  HB  VAL A  69      15.692  70.859  25.374   1.0   0.0           H
ATOM   1099 HG11 VAL A  69      14.350  72.204  23.827   1.0   0.0           H
ATOM   1100 HG12 VAL A  69      14.075  73.243  24.768   1.0   0.0           H
ATOM   1101 HG13 VAL A  69      13.548  71.920  24.974   1.0   0.0           H
ATOM   1102 HG21 VAL A  69      16.818  72.482  23.566   1.0   0.0           H
ATOM   1103 HG22 VAL A  69      17.897  72.388  24.522   1.0   0.0           H
ATOM   1104 HG23 VAL A  69      17.047  73.584  24.472   1.0   0.0           H
ATOM   1105  N   ALA A  70      13.935  71.960  28.138   1.0   0.0           N
ATOM   1106  CA  ALA A  70      12.837  71.327  28.854   1.0   0.0           C
ATOM   1107  CB  ALA A  70      13.127  71.312  30.331   1.0   0.0           C
ATOM   1108  C   ALA A  70      11.555  72.086  28.570   1.0   0.0           C
ATOM   1109  O   ALA A  70      11.574  73.287  28.351   1.0   0.0           O
ATOM   1110  H   ALA A  70      13.742  71.970  27.157   1.0   0.0           H
ATOM   1111  HA  ALA A  70      12.568  70.559  28.272   1.0   0.0           H
ATOM   1112  HB1 ALA A  70      13.972  70.812  30.518   1.0   0.0           H
ATOM   1113  HB2 ALA A  70      12.626  70.832  31.051   1.0   0.0           H
ATOM   1114  HB3 ALA A  70      13.396  72.080  30.913   1.0   0.0           H
ATOM   1115  N   VAL A  71      10.438  71.380  28.570   1.0   0.0           N
ATOM   1116  CA  VAL A  71       9.199  71.912  28.034   1.0   0.0           C
ATOM   1117  CB  VAL A  71       9.122  71.584  26.532   1.0   0.0           C
ATOM   1118  CG1 VAL A  71       7.683  71.459  26.051   1.0   0.0           C
ATOM   1119  CG2 VAL A  71       9.883  72.619  25.721   1.0   0.0           C
ATOM   1120  C   VAL A  71       8.030  71.256  28.732   1.0   0.0           C
ATOM   1121  O   VAL A  71       8.051  70.044  28.900   1.0   0.0           O
ATOM   1122  H   VAL A  71      11.211  71.814  28.108   1.0   0.0           H
ATOM   1123  HA  VAL A  71       9.047  72.810  28.446   1.0   0.0           H
ATOM   1124  HB  VAL A  71       9.355  70.624  26.684   1.0   0.0           H
ATOM   1125 HG11 VAL A  71       7.633  71.246  25.075   1.0   0.0           H
ATOM   1126 HG12 VAL A  71       6.973  72.156  25.945   1.0   0.0           H
ATOM   1127 HG13 VAL A  71       6.993  70.763  26.248   1.0   0.0           H
ATOM   1128 HG21 VAL A  71       9.833  72.406  24.745   1.0   0.0           H
ATOM   1129 HG22 VAL A  71      10.871  72.771  25.683   1.0   0.0           H
ATOM   1130 HG23 VAL A  71       9.650  73.579  25.569   1.0   0.0           H
ATOM   1131  N   ILE A  72       7.004  72.027  29.106   1.0   0.0           N
ATOM   1132  CA  ILE A  72       5.795  71.435  29.690   1.0   0.0           C
ATOM   1133  CB  ILE A  72       5.209  72.278  30.836   1.0   0.0           C
ATOM   1134  CG1 ILE A  72       6.269  72.568  31.900   1.0   0.0           C
ATOM   1135  CD1 ILE A  72       6.845  73.965  31.813   1.0   0.0           C
ATOM   1136  CG2 ILE A  72       4.051  71.553  31.498   1.0   0.0           C
ATOM   1137  C   ILE A  72       4.725  71.210  28.615   1.0   0.0           C
ATOM   1138  O   ILE A  72       4.518  72.046  27.745   1.0   0.0           O
ATOM   1139  H   ILE A  72       7.702  72.174  29.807   1.0   0.0           H
ATOM   1140  HA  ILE A  72       6.055  70.470  29.716   1.0   0.0           H
ATOM   1141  HB  ILE A  72       4.773  72.906  30.191   1.0   0.0           H
ATOM   1142 HG12 ILE A  72       5.676  72.125  32.573   1.0   0.0           H
ATOM   1143 HG13 ILE A  72       6.357  71.581  32.031   1.0   0.0           H
ATOM   1144 HD11 ILE A  72       7.538  74.155  32.509   1.0   0.0           H
ATOM   1145 HD12 ILE A  72       6.454  74.867  31.995   1.0   0.0           H
ATOM   1146 HD13 ILE A  72       7.438  74.408  31.140   1.0   0.0           H
ATOM   1147 HG21 ILE A  72       3.670  72.101  32.243   1.0   0.0           H
ATOM   1148 HG22 ILE A  72       4.075  70.728  32.063   1.0   0.0           H
ATOM   1149 HG23 ILE A  72       3.144  71.357  31.124   1.0   0.0           H
ATOM   1150  N   LEU A  73       4.073  70.052  28.682   1.0   0.0           N
ATOM   1151  CA  LEU A  73       3.067  69.620  27.704   1.0   0.0           C
ATOM   1152  CB  LEU A  73       3.439  68.245  27.139   1.0   0.0           C
ATOM   1153  CG  LEU A  73       4.859  68.142  26.575   1.0   0.0           C
ATOM   1154  CD1 LEU A  73       5.125  66.735  26.063   1.0   0.0           C
ATOM   1155  CD2 LEU A  73       5.093  69.173  25.478   1.0   0.0           C
ATOM   1156  C   LEU A  73       1.670  69.558  28.307   1.0   0.0           C
ATOM   1157  O   LEU A  73       0.678  69.841  27.640   1.0   0.0           O
ATOM   1158  H   LEU A  73       4.990  70.093  28.286   1.0   0.0           H
ATOM   1159  HA  LEU A  73       2.878  70.452  27.183   1.0   0.0           H
ATOM   1160  HB2 LEU A  73       2.970  67.867  27.937   1.0   0.0           H
ATOM   1161  HB3 LEU A  73       2.453  68.076  27.141   1.0   0.0           H
ATOM   1162  HG  LEU A  73       5.248  68.095  27.495   1.0   0.0           H
ATOM   1163 HD11 LEU A  73       6.052  66.668  25.695   1.0   0.0           H
ATOM   1164 HD12 LEU A  73       4.725  66.283  25.266   1.0   0.0           H
ATOM   1165 HD13 LEU A  73       5.224  65.894  26.595   1.0   0.0           H
ATOM   1166 HD21 LEU A  73       4.918  70.098  25.815   1.0   0.0           H
ATOM   1167 HD22 LEU A  73       4.561  69.338  24.648   1.0   0.0           H
ATOM   1168 HD23 LEU A  73       5.954  69.454  25.053   1.0   0.0           H
ATOM   1169  N   THR A  74       1.584  69.160  29.565   1.0   0.0           N
ATOM   1170  CA  THR A  74       0.346  69.310  30.294   1.0   0.0           C
ATOM   1171  CB  THR A  74      -0.619  68.140  30.031   1.0   0.0           C
ATOM   1172  OG1 THR A  74      -1.905  68.455  30.578   1.0   0.0           O
ATOM   1173  CG2 THR A  74      -0.100  66.852  30.641   1.0   0.0           C
ATOM   1174  C   THR A  74       0.645  69.464  31.776   1.0   0.0           C
ATOM   1175  O   THR A  74       1.618  68.903  32.301   1.0   0.0           O
ATOM   1176  H   THR A  74       1.387  69.059  28.590   1.0   0.0           H
ATOM   1177  HA  THR A  74       0.204  70.296  30.208   1.0   0.0           H
ATOM   1178  HB  THR A  74      -0.917  68.285  29.088   1.0   0.0           H
ATOM   1179  HG1 THR A  74      -2.532  67.695  30.407   1.0   0.0           H
ATOM   1180 HG21 THR A  74      -0.727  66.092  30.470   1.0   0.0           H
ATOM   1181 HG22 THR A  74      -0.012  66.599  31.605   1.0   0.0           H
ATOM   1182 HG23 THR A  74       0.689  66.298  30.373   1.0   0.0           H
ATOM   1183  N   HIS A  75      -0.202  70.244  32.433   1.0   0.0           N
ATOM   1184  CA  HIS A  75      -0.019  70.631  33.820   1.0   0.0           C
ATOM   1185  CB  HIS A  75       1.102  71.663  33.914   1.0   0.0           C
ATOM   1186  CG  HIS A  75       1.756  71.735  35.256   1.0   0.0           C
ATOM   1187  ND1 HIS A  75       1.366  72.626  36.230   1.0   0.0           N
ATOM   1188  CE1 HIS A  75       2.129  72.475  37.296   1.0   0.0           C
ATOM   1189  NE2 HIS A  75       3.005  71.518  37.049   1.0   0.0           N
ATOM   1190  CD2 HIS A  75       2.797  71.043  35.776   1.0   0.0           C
ATOM   1191  C   HIS A  75      -1.343  71.252  34.241   1.0   0.0           C
ATOM   1192  O   HIS A  75      -1.982  71.935  33.432   1.0   0.0           O
ATOM   1193  H   HIS A  75       0.668  69.836  32.156   1.0   0.0           H
ATOM   1194  HA  HIS A  75      -0.161  69.822  34.391   1.0   0.0           H
ATOM   1195  HB2 HIS A  75       1.421  71.345  33.021   1.0   0.0           H
ATOM   1196  HB3 HIS A  75       0.718  72.087  33.094   1.0   0.0           H
ATOM   1197  HE1 HIS A  75       2.056  72.996  38.146   1.0   0.0           H
ATOM   1198  HE2 HIS A  75       3.705  71.192  37.684   1.0   0.0           H
ATOM   1199  HD2 HIS A  75       3.317  70.321  35.319   1.0   0.0           H
ATOM   1200  N   PRO A  76      -1.783  71.010  35.481   1.0   0.0           N
ATOM   1201  CA  PRO A  76      -3.049  71.630  35.901   1.0   0.0           C
ATOM   1202  CB  PRO A  76      -3.367  70.921  37.217   1.0   0.0           C
ATOM   1203  CG  PRO A  76      -2.591  69.652  37.165   1.0   0.0           C
ATOM   1204  CD  PRO A  76      -1.327  69.994  36.439   1.0   0.0           C
ATOM   1205  C   PRO A  76      -2.973  73.149  36.105   1.0   0.0           C
ATOM   1206  O   PRO A  76      -3.963  73.850  35.884   1.0   0.0           O
ATOM   1207  HA  PRO A  76      -3.623  71.733  35.089   1.0   0.0           H
ATOM   1208  HB2 PRO A  76      -3.092  71.622  37.875   1.0   0.0           H
ATOM   1209  HB3 PRO A  76      -4.280  71.222  37.492   1.0   0.0           H
ATOM   1210  HG2 PRO A  76      -2.550  69.432  38.140   1.0   0.0           H
ATOM   1211  HG3 PRO A  76      -3.270  68.919  37.195   1.0   0.0           H
ATOM   1212  HD2 PRO A  76      -0.728  70.359  37.152   1.0   0.0           H
ATOM   1213  HD3 PRO A  76      -0.707  69.218  36.327   1.0   0.0           H
ATOM   1214  N   ALA A  77      -1.813  73.641  36.531   1.0   0.0           N
ATOM   1215  CA  ALA A  77      -1.573  75.089  36.663   1.0   0.0           C
ATOM   1216  CB  ALA A  77      -0.242  75.345  37.353   1.0   0.0           C
ATOM   1217  C   ALA A  77      -1.623  75.862  35.344   1.0   0.0           C
ATOM   1218  O   ALA A  77      -1.944  77.051  35.330   1.0   0.0           O
ATOM   1219  H   ALA A  77      -2.688  73.473  36.077   1.0   0.0           H
ATOM   1220  HA  ALA A  77      -2.473  75.406  36.961   1.0   0.0           H
ATOM   1221  HB1 ALA A  77      -0.209  74.840  38.215   1.0   0.0           H
ATOM   1222  HB2 ALA A  77       0.064  76.203  37.766   1.0   0.0           H
ATOM   1223  HB3 ALA A  77       0.658  75.028  37.055   1.0   0.0           H
ATOM   1224  N   PHE A  78      -1.304  75.191  34.242   1.0   0.0           N
ATOM   1225  CA  PHE A  78      -1.226  75.846  32.932   1.0   0.0           C
ATOM   1226  CB  PHE A  78      -0.147  75.192  32.069   1.0   0.0           C
ATOM   1227  CG  PHE A  78       1.259  75.533  32.495   1.0   0.0           C
ATOM   1228  CD1 PHE A  78       1.687  75.296  33.803   1.0   0.0           C
ATOM   1229  CE1 PHE A  78       2.983  75.597  34.197   1.0   0.0           C
ATOM   1230  CZ  PHE A  78       3.874  76.135  33.284   1.0   0.0           C
ATOM   1231  CE2 PHE A  78       3.467  76.371  31.979   1.0   0.0           C
ATOM   1232  CD2 PHE A  78       2.166  76.072  31.591   1.0   0.0           C
ATOM   1233  C   PHE A  78      -2.582  75.829  32.228   1.0   0.0           C
ATOM   1234  O   PHE A  78      -2.789  75.090  31.265   1.0   0.0           O
ATOM   1235  H   PHE A  78      -0.417  75.202  34.703   1.0   0.0           H
ATOM   1236  HA  PHE A  78      -1.342  76.806  33.185   1.0   0.0           H
ATOM   1237  HB2 PHE A  78      -0.675  74.344  32.104   1.0   0.0           H
ATOM   1238  HB3 PHE A  78      -0.849  75.042  31.373   1.0   0.0           H
ATOM   1239  HD1 PHE A  78       1.050  74.903  34.466   1.0   0.0           H
ATOM   1240  HE1 PHE A  78       3.273  75.426  35.139   1.0   0.0           H
ATOM   1241  HZ  PHE A  78       4.809  76.353  33.565   1.0   0.0           H
ATOM   1242  HE2 PHE A  78       4.110  76.756  31.317   1.0   0.0           H
ATOM   1243  HD2 PHE A  78       1.879  76.247  30.649   1.0   0.0           H
ATOM   1244  N   THR A  79      -3.507  76.634  32.758   1.0   0.0           N
ATOM   1245  CA  THR A  79      -4.843  76.824  32.176   1.0   0.0           C
ATOM   1246  CB  THR A  79      -5.948  76.057  32.954   1.0   0.0           C
ATOM   1247  OG1 THR A  79      -5.746  76.198  34.365   1.0   0.0           O
ATOM   1248  CG2 THR A  79      -5.945  74.564  32.576   1.0   0.0           C
ATOM   1249  C   THR A  79      -5.194  78.311  32.126   1.0   0.0           C
ATOM   1250  O   THR A  79      -4.801  79.085  33.000   1.0   0.0           O
ATOM   1251  H   THR A  79      -3.277  75.661  32.791   1.0   0.0           H
ATOM   1252  HA  THR A  79      -4.644  76.779  31.197   1.0   0.0           H
ATOM   1253  HB  THR A  79      -6.725  76.686  32.945   1.0   0.0           H
ATOM   1254  HG1 THR A  79      -6.457  75.704  34.866   1.0   0.0           H
ATOM   1255 HG21 THR A  79      -6.656  74.070  33.077   1.0   0.0           H
ATOM   1256 HG22 THR A  79      -5.265  73.861  32.786   1.0   0.0           H
ATOM   1257 HG23 THR A  79      -6.208  74.131  31.714   1.0   0.0           H
ATOM   1258  N   GLY A  80      -5.931  78.696  31.089   1.0   0.0           N
ATOM   1259  CA  GLY A  80      -6.305  80.085  30.889   1.0   0.0           C
ATOM   1260  C   GLY A  80      -5.121  80.887  30.397   1.0   0.0           C
ATOM   1261  O   GLY A  80      -4.563  80.606  29.328   1.0   0.0           O
ATOM   1262  H   GLY A  80      -6.714  78.166  31.414   1.0   0.0           H
ATOM   1263  HA2 GLY A  80      -7.134  79.839  30.387   1.0   0.0           H
ATOM   1264  HA3 GLY A  80      -7.105  80.024  31.485   1.0   0.0           H
ATOM   1265  N   GLN A  81      -4.719  81.868  31.198   1.0   0.0           N
ATOM   1266  CA  GLN A  81      -3.621  82.761  30.835   1.0   0.0           C
ATOM   1267  CB  GLN A  81      -3.392  83.779  31.951   1.0   0.0           C
ATOM   1268  CG  GLN A  81      -2.176  84.669  31.754   1.0   0.0           C
ATOM   1269  CD  GLN A  81      -2.308  85.968  32.535   1.0   0.0           C
ATOM   1270  OE1 GLN A  81      -2.498  87.054  31.978   1.0   0.0           O
ATOM   1271  NE2 GLN A  81      -2.232  85.848  33.853   1.0   0.0           N
ATOM   1272  C   GLN A  81      -2.311  82.025  30.538   1.0   0.0           C
ATOM   1273  O   GLN A  81      -1.565  82.424  29.640   1.0   0.0           O
ATOM   1274  H   GLN A  81      -5.574  82.349  31.392   1.0   0.0           H
ATOM   1275  HA  GLN A  81      -3.806  82.957  29.872   1.0   0.0           H
ATOM   1276  HB2 GLN A  81      -4.379  83.939  31.932   1.0   0.0           H
ATOM   1277  HB3 GLN A  81      -4.010  83.213  32.497   1.0   0.0           H
ATOM   1278  HG2 GLN A  81      -1.558  83.897  31.903   1.0   0.0           H
ATOM   1279  HG3 GLN A  81      -1.914  84.181  30.922   1.0   0.0           H
ATOM   1280 HE21 GLN A  81      -2.078  84.969  34.304   1.0   0.0           H
ATOM   1281 HE22 GLN A  81      -2.332  86.667  34.417   1.0   0.0           H
ATOM   1282  N   TYR A  82      -2.047  80.956  31.293   1.0   0.0           N
ATOM   1283  CA  TYR A  82      -0.747  80.279  31.270   1.0   0.0           C
ATOM   1284  CB  TYR A  82      -0.380  79.805  32.680   1.0   0.0           C
ATOM   1285  CG  TYR A  82      -0.716  80.853  33.688   1.0   0.0           C
ATOM   1286  CD1 TYR A  82      -0.175  82.130  33.576   1.0   0.0           C
ATOM   1287  CE1 TYR A  82      -0.499  83.125  34.473   1.0   0.0           C
ATOM   1288  CZ  TYR A  82      -1.385  82.852  35.494   1.0   0.0           C
ATOM   1289  OH  TYR A  82      -1.693  83.855  36.381   1.0   0.0           O
ATOM   1290  CE2 TYR A  82      -1.950  81.590  35.622   1.0   0.0           C
ATOM   1291  CD2 TYR A  82      -1.619  80.600  34.713   1.0   0.0           C
ATOM   1292  C   TYR A  82      -0.690  79.119  30.288   1.0   0.0           C
ATOM   1293  O   TYR A  82       0.364  78.500  30.134   1.0   0.0           O
ATOM   1294  H   TYR A  82      -2.084  81.719  31.939   1.0   0.0           H
ATOM   1295  HA  TYR A  82      -0.227  80.876  30.659   1.0   0.0           H
ATOM   1296  HB2 TYR A  82      -0.821  78.916  32.556   1.0   0.0           H
ATOM   1297  HB3 TYR A  82       0.275  79.119  32.362   1.0   0.0           H
ATOM   1298  HD1 TYR A  82       0.461  82.330  32.831   1.0   0.0           H
ATOM   1299  HE1 TYR A  82      -0.096  84.036  34.384   1.0   0.0           H
ATOM   1300  HH  TYR A  82      -1.286  84.764  36.289   1.0   0.0           H
ATOM   1301  HE2 TYR A  82      -2.591  81.398  36.365   1.0   0.0           H
ATOM   1302  HD2 TYR A  82      -2.034  79.694  34.795   1.0   0.0           H
ATOM   1303  N   GLU A  83      -1.803  78.841  29.609   1.0   0.0           N
ATOM   1304  CA  GLU A  83      -1.822  77.791  28.594   1.0   0.0           C
ATOM   1305  CB  GLU A  83      -3.211  77.630  27.976   1.0   0.0           C
ATOM   1306  CG  GLU A  83      -3.904  76.360  28.415   1.0   0.0           C
ATOM   1307  CD  GLU A  83      -5.376  76.349  28.066   1.0   0.0           C
ATOM   1308  OE1 GLU A  83      -5.690  76.402  26.854   1.0   0.0           O
ATOM   1309  OE2 GLU A  83      -6.202  76.290  29.012   1.0   0.0           O
ATOM   1310  C   GLU A  83      -0.813  78.043  27.491   1.0   0.0           C
ATOM   1311  O   GLU A  83      -0.201  77.112  26.986   1.0   0.0           O
ATOM   1312  H   GLU A  83      -2.469  78.675  30.337   1.0   0.0           H
ATOM   1313  HA  GLU A  83      -1.279  77.073  29.028   1.0   0.0           H
ATOM   1314  HB2 GLU A  83      -3.403  78.590  28.178   1.0   0.0           H
ATOM   1315  HB3 GLU A  83      -2.963  78.298  27.274   1.0   0.0           H
ATOM   1316  HG2 GLU A  83      -3.139  75.825  28.056   1.0   0.0           H
ATOM   1317  HG3 GLU A  83      -3.151  76.176  29.047   1.0   0.0           H
ATOM   1318  N   LYS A  84      -0.634  79.307  27.127   1.0   0.0           N
ATOM   1319  CA  LYS A  84       0.298  79.666  26.060   1.0   0.0           C
ATOM   1320  CB  LYS A  84       0.052  81.107  25.586   1.0   0.0           C
ATOM   1321  CG  LYS A  84       0.366  82.207  26.592   1.0   0.0           C
ATOM   1322  CD  LYS A  84      -0.260  83.521  26.150   1.0   0.0           C
ATOM   1323  CE  LYS A  84       0.367  84.718  26.840   1.0   0.0           C
ATOM   1324  NZ  LYS A  84       0.056  84.759  28.290   1.0   0.0           N
ATOM   1325  C   LYS A  84       1.781  79.443  26.416   1.0   0.0           C
ATOM   1326  O   LYS A  84       2.645  79.605  25.550   1.0   0.0           O
ATOM   1327  H   LYS A  84      -1.596  79.452  26.896   1.0   0.0           H
ATOM   1328  HA  LYS A  84       0.270  78.811  25.542   1.0   0.0           H
ATOM   1329  HB2 LYS A  84       0.444  80.788  24.723   1.0   0.0           H
ATOM   1330  HB3 LYS A  84      -0.481  80.652  24.872   1.0   0.0           H
ATOM   1331  HG2 LYS A  84       0.110  81.635  27.371   1.0   0.0           H
ATOM   1332  HG3 LYS A  84       1.149  81.722  26.981   1.0   0.0           H
ATOM   1333  HD2 LYS A  84      -0.219  83.222  25.197   1.0   0.0           H
ATOM   1334  HD3 LYS A  84      -1.060  83.028  25.808   1.0   0.0           H
ATOM   1335  HE2 LYS A  84       1.236  84.548  26.376   1.0   0.0           H
ATOM   1336  HE3 LYS A  84       0.465  85.195  25.966   1.0   0.0           H
ATOM   1337  HZ1 LYS A  84       0.469  85.548  28.745   1.0   0.0           H
ATOM   1338  HZ2 LYS A  84       0.360  84.156  29.028   1.0   0.0           H
ATOM   1339  HZ3 LYS A  84      -0.813  84.929  28.754   1.0   0.0           H
ATOM   1340  N   TYR A  85       2.078  79.079  27.671   1.0   0.0           N
ATOM   1341  CA  TYR A  85       3.448  78.730  28.079   1.0   0.0           C
ATOM   1342  CB  TYR A  85       3.776  79.338  29.450   1.0   0.0           C
ATOM   1343  CG  TYR A  85       3.765  80.857  29.469   1.0   0.0           C
ATOM   1344  CD1 TYR A  85       4.308  81.602  28.418   1.0   0.0           C
ATOM   1345  CE1 TYR A  85       4.304  82.985  28.435   1.0   0.0           C
ATOM   1346  CZ  TYR A  85       3.769  83.649  29.520   1.0   0.0           C
ATOM   1347  OH  TYR A  85       3.772  85.023  29.543   1.0   0.0           O
ATOM   1348  CE2 TYR A  85       3.232  82.942  30.586   1.0   0.0           C
ATOM   1349  CD2 TYR A  85       3.232  81.553  30.555   1.0   0.0           C
ATOM   1350  C   TYR A  85       3.730  77.218  28.067   1.0   0.0           C
ATOM   1351  O   TYR A  85       4.836  76.783  28.399   1.0   0.0           O
ATOM   1352  H   TYR A  85       1.895  80.062  27.679   1.0   0.0           H
ATOM   1353  HA  TYR A  85       3.916  78.853  27.204   1.0   0.0           H
ATOM   1354  HB2 TYR A  85       3.179  78.646  29.856   1.0   0.0           H
ATOM   1355  HB3 TYR A  85       4.155  78.446  29.697   1.0   0.0           H
ATOM   1356  HD1 TYR A  85       4.708  81.124  27.636   1.0   0.0           H
ATOM   1357  HE1 TYR A  85       4.686  83.500  27.667   1.0   0.0           H
ATOM   1358  HH  TYR A  85       4.159  85.533  28.775   1.0   0.0           H
ATOM   1359  HE2 TYR A  85       2.848  83.428  31.371   1.0   0.0           H
ATOM   1360  HD2 TYR A  85       2.844  81.044  31.323   1.0   0.0           H
ATOM   1361  N   LEU A  86       2.745  76.419  27.670   1.0   0.0           N
ATOM   1362  CA  LEU A  86       3.011  75.022  27.338   1.0   0.0           C
ATOM   1363  CB  LEU A  86       1.713  74.247  27.158   1.0   0.0           C
ATOM   1364  CG  LEU A  86       0.874  74.064  28.413   1.0   0.0           C
ATOM   1365  CD1 LEU A  86      -0.499  73.554  28.036   1.0   0.0           C
ATOM   1366  CD2 LEU A  86       1.551  73.111  29.386   1.0   0.0           C
ATOM   1367  C   LEU A  86       3.805  74.971  26.040   1.0   0.0           C
ATOM   1368  O   LEU A  86       3.623  75.815  25.175   1.0   0.0           O
ATOM   1369  H   LEU A  86       2.223  76.452  28.523   1.0   0.0           H
ATOM   1370  HA  LEU A  86       3.784  74.753  27.913   1.0   0.0           H
ATOM   1371  HB2 LEU A  86       1.620  74.766  26.308   1.0   0.0           H
ATOM   1372  HB3 LEU A  86       2.133  73.956  26.298   1.0   0.0           H
ATOM   1373  HG  LEU A  86       0.727  75.049  28.498   1.0   0.0           H
ATOM   1374 HD11 LEU A  86      -1.051  73.434  28.861   1.0   0.0           H
ATOM   1375 HD12 LEU A  86      -0.721  72.649  27.672   1.0   0.0           H
ATOM   1376 HD13 LEU A  86      -1.212  74.060  27.550   1.0   0.0           H
ATOM   1377 HD21 LEU A  86       2.459  73.448  29.635   1.0   0.0           H
ATOM   1378 HD22 LEU A  86       1.896  72.188  29.214   1.0   0.0           H
ATOM   1379 HD23 LEU A  86       1.314  72.923  30.339   1.0   0.0           H
ATOM   1380  N   GLY A  87       4.704  74.003  25.924   1.0   0.0           N
ATOM   1381  CA  GLY A  87       5.530  73.875  24.738   1.0   0.0           C
ATOM   1382  C   GLY A  87       6.677  74.872  24.673   1.0   0.0           C
ATOM   1383  O   GLY A  87       7.326  75.010  23.626   1.0   0.0           O
ATOM   1384  H   GLY A  87       3.950  73.348  25.967   1.0   0.0           H
ATOM   1385  HA2 GLY A  87       5.491  72.884  24.869   1.0   0.0           H
ATOM   1386  HA3 GLY A  87       4.777  73.353  24.336   1.0   0.0           H
ATOM   1387  N   VAL A  88       6.933  75.559  25.787   1.0   0.0           N
ATOM   1388  CA  VAL A  88       7.934  76.616  25.845   1.0   0.0           C
ATOM   1389  CB  VAL A  88       7.308  77.950  26.319   1.0   0.0           C
ATOM   1390  CG1 VAL A  88       8.357  79.050  26.386   1.0   0.0           C
ATOM   1391  CG2 VAL A  88       6.169  78.364  25.393   1.0   0.0           C
ATOM   1392  C   VAL A  88       9.078  76.190  26.773   1.0   0.0           C
ATOM   1393  O   VAL A  88       8.850  75.557  27.811   1.0   0.0           O
ATOM   1394  H   VAL A  88       6.187  75.837  25.182   1.0   0.0           H
ATOM   1395  HA  VAL A  88       8.487  76.426  25.034   1.0   0.0           H
ATOM   1396  HB  VAL A  88       7.254  77.638  27.267   1.0   0.0           H
ATOM   1397 HG11 VAL A  88       7.953  79.912  26.692   1.0   0.0           H
ATOM   1398 HG12 VAL A  88       8.840  79.504  25.637   1.0   0.0           H
ATOM   1399 HG13 VAL A  88       9.126  79.148  27.018   1.0   0.0           H
ATOM   1400 HG21 VAL A  88       5.765  79.226  25.699   1.0   0.0           H
ATOM   1401 HG22 VAL A  88       5.273  77.934  25.281   1.0   0.0           H
ATOM   1402 HG23 VAL A  88       6.223  78.676  24.445   1.0   0.0           H
ATOM   1403  N   ASP A  89      10.303  76.539  26.377   1.0   0.0           N
ATOM   1404  CA  ASP A  89      11.508  76.209  27.132   1.0   0.0           C
ATOM   1405  CB  ASP A  89      12.758  76.582  26.322   1.0   0.0           C
ATOM   1406  CG  ASP A  89      14.059  76.171  27.004   1.0   0.0           C
ATOM   1407  OD1 ASP A  89      14.074  75.197  27.792   1.0   0.0           O
ATOM   1408  OD2 ASP A  89      15.083  76.834  26.743   1.0   0.0           O
ATOM   1409  C   ASP A  89      11.468  76.963  28.458   1.0   0.0           C
ATOM   1410  O   ASP A  89      10.960  78.078  28.516   1.0   0.0           O
ATOM   1411  H   ASP A  89       9.489  76.296  26.905   1.0   0.0           H
ATOM   1412  HA  ASP A  89      11.278  75.348  27.586   1.0   0.0           H
ATOM   1413  HB2 ASP A  89      12.297  76.249  25.500   1.0   0.0           H
ATOM   1414  HB3 ASP A  89      12.194  77.224  25.803   1.0   0.0           H
ATOM   1415  N   VAL A  90      11.991  76.360  29.520   1.0   0.0           N
ATOM   1416  CA  VAL A  90      11.875  76.963  30.851   1.0   0.0           C
ATOM   1417  CB  VAL A  90      11.526  75.919  31.933   1.0   0.0           C
ATOM   1418  CG1 VAL A  90      10.203  75.255  31.586   1.0   0.0           C
ATOM   1419  CG2 VAL A  90      12.633  74.883  32.093   1.0   0.0           C
ATOM   1420  C   VAL A  90      13.094  77.788  31.253   1.0   0.0           C
ATOM   1421  O   VAL A  90      13.184  78.252  32.386   1.0   0.0           O
ATOM   1422  H   VAL A  90      11.192  75.819  29.257   1.0   0.0           H
ATOM   1423  HA  VAL A  90      11.384  77.789  30.575   1.0   0.0           H
ATOM   1424  HB  VAL A  90      11.227  76.642  32.556   1.0   0.0           H
ATOM   1425 HG11 VAL A  90       9.977  74.579  32.287   1.0   0.0           H
ATOM   1426 HG12 VAL A  90      10.032  74.629  30.825   1.0   0.0           H
ATOM   1427 HG13 VAL A  90       9.299  75.682  31.605   1.0   0.0           H
ATOM   1428 HG21 VAL A  90      12.407  74.207  32.794   1.0   0.0           H
ATOM   1429 HG22 VAL A  90      13.556  74.981  32.464   1.0   0.0           H
ATOM   1430 HG23 VAL A  90      12.932  74.160  31.470   1.0   0.0           H
ATOM   1431  N   ALA A  91      14.026  77.977  30.326   1.0   0.0           N
ATOM   1432  CA  ALA A  91      15.014  79.029  30.478   1.0   0.0           C
ATOM   1433  CB  ALA A  91      15.861  79.148  29.221   1.0   0.0           C
ATOM   1434  C   ALA A  91      14.276  80.336  30.754   1.0   0.0           C
ATOM   1435  O   ALA A  91      14.668  81.099  31.641   1.0   0.0           O
ATOM   1436  H   ALA A  91      13.469  77.899  31.153   1.0   0.0           H
ATOM   1437  HA  ALA A  91      15.319  78.952  31.427   1.0   0.0           H
ATOM   1438  HB1 ALA A  91      16.345  78.292  29.040   1.0   0.0           H
ATOM   1439  HB2 ALA A  91      16.687  79.694  29.078   1.0   0.0           H
ATOM   1440  HB3 ALA A  91      15.556  79.225  28.272   1.0   0.0           H
ATOM   1441  N   MET A  92      13.181  80.560  30.023   1.0   0.0           N
ATOM   1442  CA  MET A  92      12.424  81.810  30.144   1.0   0.0           C
ATOM   1443  CB  MET A  92      11.090  81.998  29.465   1.0   0.0           C
ATOM   1444  CG  MET A  92      11.141  82.803  28.142   1.0   0.0           C
ATOM   1445  SD  MET A  92      12.119  82.021  26.607   1.0   0.0           S
ATOM   1446  CE  MET A  92      11.243  80.275  26.546   1.0   0.0           C
ATOM   1447  C   MET A  92      11.955  81.999  31.540   1.0   0.0           C
ATOM   1448  O   MET A  92      12.158  83.053  32.111   1.0   0.0           O
ATOM   1449  H   MET A  92      13.497  80.433  29.083   1.0   0.0           H
ATOM   1450  HA  MET A  92      12.962  82.613  29.887   1.0   0.0           H
ATOM   1451  HB2 MET A  92      10.916  81.043  29.706   1.0   0.0           H
ATOM   1452  HB3 MET A  92      10.709  81.726  30.348   1.0   0.0           H
ATOM   1453  HG2 MET A  92      10.264  83.030  28.565   1.0   0.0           H
ATOM   1454  HG3 MET A  92      10.709  83.374  28.840   1.0   0.0           H
ATOM   1455  HE1 MET A  92      11.737  79.880  25.771   1.0   0.0           H
ATOM   1456  HE2 MET A  92      10.309  80.283  26.189   1.0   0.0           H
ATOM   1457  HE3 MET A  92      11.505  79.645  27.277   1.0   0.0           H
ATOM   1458  N   LEU A  93      11.250  81.005  32.067   1.0   0.0           N
ATOM   1459  CA  LEU A  93      10.719  81.089  33.426   1.0   0.0           C
ATOM   1460  CB  LEU A  93       9.818  79.892  33.728   1.0   0.0           C
ATOM   1461  CG  LEU A  93       8.424  79.974  33.084   1.0   0.0           C
ATOM   1462  CD1 LEU A  93       8.013  78.673  32.391   1.0   0.0           C
ATOM   1463  CD2 LEU A  93       7.387  80.407  34.117   1.0   0.0           C
ATOM   1464  C   LEU A  93      11.836  81.191  34.458   1.0   0.0           C
ATOM   1465  O   LEU A  93      11.606  81.622  35.585   1.0   0.0           O
ATOM   1466  H   LEU A  93      10.517  80.938  31.390   1.0   0.0           H
ATOM   1467  HA  LEU A  93      10.505  82.063  33.501   1.0   0.0           H
ATOM   1468  HB2 LEU A  93      10.621  79.316  33.573   1.0   0.0           H
ATOM   1469  HB3 LEU A  93      10.455  79.688  34.471   1.0   0.0           H
ATOM   1470  HG  LEU A  93       8.766  80.490  32.299   1.0   0.0           H
ATOM   1471 HD11 LEU A  93       7.106  78.726  31.972   1.0   0.0           H
ATOM   1472 HD12 LEU A  93       7.775  77.771  32.752   1.0   0.0           H
ATOM   1473 HD13 LEU A  93       8.350  78.222  31.565   1.0   0.0           H
ATOM   1474 HD21 LEU A  93       7.656  81.257  34.570   1.0   0.0           H
ATOM   1475 HD22 LEU A  93       7.147  79.992  34.995   1.0   0.0           H
ATOM   1476 HD23 LEU A  93       6.456  80.748  33.984   1.0   0.0           H
ATOM   1477  N   ARG A  94      13.043  80.787  34.067   1.0   0.0           N
ATOM   1478  CA  ARG A  94      14.247  81.116  34.822   1.0   0.0           C
ATOM   1479  CB  ARG A  94      14.378  82.650  34.936   1.0   0.0           C
ATOM   1480  CG  ARG A  94      15.769  83.209  34.664   1.0   0.0           C
ATOM   1481  CD  ARG A  94      16.422  83.788  35.911   1.0   0.0           C
ATOM   1482  NE  ARG A  94      15.924  85.131  36.223   1.0   0.0           N
ATOM   1483  CZ  ARG A  94      16.242  86.242  35.551   1.0   0.0           C
ATOM   1484  NH1 ARG A  94      17.061  86.200  34.501   1.0   0.0           N
ATOM   1485  NH2 ARG A  94      15.731  87.410  35.927   1.0   0.0           N
ATOM   1486  C   ARG A  94      14.197  80.422  36.186   1.0   0.0           C
ATOM   1487  O   ARG A  94      14.372  81.041  37.233   1.0   0.0           O
ATOM   1488  H   ARG A  94      12.958  79.793  33.993   1.0   0.0           H
ATOM   1489  HA  ARG A  94      14.882  80.410  34.509   1.0   0.0           H
ATOM   1490  HB2 ARG A  94      13.497  82.682  34.464   1.0   0.0           H
ATOM   1491  HB3 ARG A  94      13.508  82.588  35.425   1.0   0.0           H
ATOM   1492  HG2 ARG A  94      15.981  82.394  34.125   1.0   0.0           H
ATOM   1493  HG3 ARG A  94      15.553  83.240  33.688   1.0   0.0           H
ATOM   1494  HD2 ARG A  94      16.285  82.933  36.411   1.0   0.0           H
ATOM   1495  HD3 ARG A  94      17.201  83.165  35.834   1.0   0.0           H
ATOM   1496  HE  ARG A  94      15.299  85.223  36.998   1.0   0.0           H
ATOM   1497 HH11 ARG A  94      17.445  85.321  34.218   1.0   0.0           H
ATOM   1498 HH12 ARG A  94      17.299  87.031  33.998   1.0   0.0           H
ATOM   1499 HH21 ARG A  94      15.116  87.442  36.715   1.0   0.0           H
ATOM   1500 HH22 ARG A  94      15.969  88.241  35.424   1.0   0.0           H
ATOM   1501  N   TRP A  95      13.942  79.117  36.151   1.0   0.0           N
ATOM   1502  CA  TRP A  95      13.825  78.317  37.369   1.0   0.0           C
ATOM   1503  CB  TRP A  95      13.090  76.998  37.085   1.0   0.0           C
ATOM   1504  CG  TRP A  95      11.629  77.167  36.754   1.0   0.0           C
ATOM   1505  CD1 TRP A  95      10.895  78.322  36.806   1.0   0.0           C
ATOM   1506  NE1 TRP A  95       9.591  78.074  36.448   1.0   0.0           N
ATOM   1507  CE2 TRP A  95       9.452  76.742  36.169   1.0   0.0           C
ATOM   1508  CD2 TRP A  95      10.715  76.135  36.359   1.0   0.0           C
ATOM   1509  CE3 TRP A  95      10.843  74.759  36.134   1.0   0.0           C
ATOM   1510  CZ3 TRP A  95       9.718  74.042  35.730   1.0   0.0           C
ATOM   1511  CH2 TRP A  95       8.473  74.677  35.549   1.0   0.0           C
ATOM   1512  CZ2 TRP A  95       8.321  76.019  35.763   1.0   0.0           C
ATOM   1513  C   TRP A  95      15.174  78.034  38.026   1.0   0.0           C
ATOM   1514  O   TRP A  95      15.222  77.599  39.174   1.0   0.0           O
ATOM   1515  H   TRP A  95      13.059  79.302  35.721   1.0   0.0           H
ATOM   1516  HA  TRP A  95      13.590  79.021  38.040   1.0   0.0           H
ATOM   1517  HB2 TRP A  95      13.883  76.718  36.544   1.0   0.0           H
ATOM   1518  HB3 TRP A  95      13.850  76.535  37.542   1.0   0.0           H
ATOM   1519  HD1 TRP A  95      11.256  79.217  37.067   1.0   0.0           H
ATOM   1520  HE1 TRP A  95       8.862  78.757  36.399   1.0   0.0           H
ATOM   1521  HE3 TRP A  95      11.723  74.300  36.260   1.0   0.0           H
ATOM   1522  HZ3 TRP A  95       9.796  73.059  35.566   1.0   0.0           H
ATOM   1523  HH2 TRP A  95       7.685  74.135  35.258   1.0   0.0           H
ATOM   1524  HZ2 TRP A  95       7.437  76.468  35.635   1.0   0.0           H
ATOM   1525  N   ASN A  96      16.257  78.242  37.286   1.0   0.0           N
ATOM   1526  CA  ASN A  96      17.603  78.256  37.861   1.0   0.0           C
ATOM   1527  CB  ASN A  96      18.628  78.639  36.792   1.0   0.0           C
ATOM   1528  CG  ASN A  96      18.123  79.722  35.854   1.0   0.0           C
ATOM   1529  OD1 ASN A  96      17.225  79.487  35.040   1.0   0.0           O
ATOM   1530  ND2 ASN A  96      18.695  80.915  35.963   1.0   0.0           N
ATOM   1531  C   ASN A  96      17.729  79.202  39.058   1.0   0.0           C
ATOM   1532  O   ASN A  96      18.498  78.947  39.989   1.0   0.0           O
ATOM   1533  H   ASN A  96      15.587  77.992  37.985   1.0   0.0           H
ATOM   1534  HA  ASN A  96      17.602  77.435  38.432   1.0   0.0           H
ATOM   1535  HB2 ASN A  96      19.340  78.650  37.494   1.0   0.0           H
ATOM   1536  HB3 ASN A  96      19.110  77.781  36.968   1.0   0.0           H
ATOM   1537 HD21 ASN A  96      19.422  81.105  36.622   1.0   0.0           H
ATOM   1538 HD22 ASN A  96      18.386  81.652  35.362   1.0   0.0           H
ATOM   1539  N   GLU A  97      16.976  80.298  39.016   1.0   0.0           N
ATOM   1540  CA  GLU A  97      16.927  81.258  40.116   1.0   0.0           C
ATOM   1541  CB  GLU A  97      16.747  82.675  39.561   1.0   0.0           C
ATOM   1542  CG  GLU A  97      17.557  83.735  40.282   1.0   0.0           C
ATOM   1543  CD  GLU A  97      18.935  83.940  39.673   1.0   0.0           C
ATOM   1544  OE1 GLU A  97      19.899  84.109  40.451   1.0   0.0           O
ATOM   1545  OE2 GLU A  97      19.062  83.943  38.425   1.0   0.0           O
ATOM   1546  C   GLU A  97      15.768  80.915  41.049   1.0   0.0           C
ATOM   1547  O   GLU A  97      15.968  80.693  42.240   1.0   0.0           O
ATOM   1548  H   GLU A  97      17.735  80.523  38.405   1.0   0.0           H
ATOM   1549  HA  GLU A  97      17.617  80.940  40.766   1.0   0.0           H
ATOM   1550  HB2 GLU A  97      16.800  82.272  38.647   1.0   0.0           H
ATOM   1551  HB3 GLU A  97      15.919  82.330  39.120   1.0   0.0           H
ATOM   1552  HG2 GLU A  97      16.747  84.321  40.303   1.0   0.0           H
ATOM   1553  HG3 GLU A  97      16.945  83.645  41.068   1.0   0.0           H
ATOM   1554  N   THR A  98      14.562  80.874  40.479   1.0   0.0           N
ATOM   1555  CA  THR A  98      13.314  80.688  41.225   1.0   0.0           C
ATOM   1556  CB  THR A  98      12.091  80.611  40.277   1.0   0.0           C
ATOM   1557  OG1 THR A  98      12.234  81.567  39.222   1.0   0.0           O
ATOM   1558  CG2 THR A  98      10.794  80.883  41.031   1.0   0.0           C
ATOM   1559  C   THR A  98      13.328  79.429  42.077   1.0   0.0           C
ATOM   1560  O   THR A  98      13.196  79.499  43.298   1.0   0.0           O
ATOM   1561  H   THR A  98      14.553  81.702  39.919   1.0   0.0           H
ATOM   1562  HA  THR A  98      13.408  81.322  41.993   1.0   0.0           H
ATOM   1563  HB  THR A  98      12.284  79.826  39.689   1.0   0.0           H
ATOM   1564  HG1 THR A  98      11.445  81.517  38.610   1.0   0.0           H
ATOM   1565 HG21 THR A  98      10.005  80.833  40.419   1.0   0.0           H
ATOM   1566 HG22 THR A  98      10.464  81.737  41.434   1.0   0.0           H
ATOM   1567 HG23 THR A  98      10.332  80.322  41.718   1.0   0.0           H
ATOM   1568  N   MET A  99      13.471  78.281  41.422   1.0   0.0           N
ATOM   1569  CA  MET A  99      13.396  76.985  42.090   1.0   0.0           C
ATOM   1570  CB  MET A  99      12.149  76.237  41.620   1.0   0.0           C
ATOM   1571  CG  MET A  99      10.838  76.960  41.927   1.0   0.0           C
ATOM   1572  SD  MET A  99       9.550  76.685  40.455   1.0   0.0           S
ATOM   1573  CE  MET A  99       8.293  75.513  41.437   1.0   0.0           C
ATOM   1574  C   MET A  99      14.617  76.200  41.719   1.0   0.0           C
ATOM   1575  O   MET A  99      14.531  75.280  40.915   1.0   0.0           O
ATOM   1576  H   MET A  99      12.656  78.805  41.670   1.0   0.0           H
ATOM   1577  HA  MET A  99      13.690  77.125  43.036   1.0   0.0           H
ATOM   1578  HB2 MET A  99      12.642  76.039  40.773   1.0   0.0           H
ATOM   1579  HB3 MET A  99      12.730  75.426  41.558   1.0   0.0           H
ATOM   1580  HG2 MET A  99      10.970  76.638  42.865   1.0   0.0           H
ATOM   1581  HG3 MET A  99      11.351  77.357  42.688   1.0   0.0           H
ATOM   1582  HE1 MET A  99       7.641  75.374  40.692   1.0   0.0           H
ATOM   1583  HE2 MET A  99       7.754  76.037  42.097   1.0   0.0           H
ATOM   1584  HE3 MET A  99       8.676  74.598  41.559   1.0   0.0           H
ATOM   1585  N   PRO A 100      15.782  76.555  42.291   1.0   0.0           N
ATOM   1586  CA  PRO A 100      17.015  75.892  41.861   1.0   0.0           C
ATOM   1587  CB  PRO A 100      18.106  76.643  42.625   1.0   0.0           C
ATOM   1588  CG  PRO A 100      17.429  77.177  43.838   1.0   0.0           C
ATOM   1589  CD  PRO A 100      15.996  77.425  43.462   1.0   0.0           C
ATOM   1590  C   PRO A 100      17.029  74.404  42.210   1.0   0.0           C
ATOM   1591  O   PRO A 100      17.511  73.595  41.414   1.0   0.0           O
ATOM   1592  HA  PRO A 100      16.952  75.693  40.883   1.0   0.0           H
ATOM   1593  HB2 PRO A 100      18.785  75.917  42.735   1.0   0.0           H
ATOM   1594  HB3 PRO A 100      18.766  76.955  41.942   1.0   0.0           H
ATOM   1595  HG2 PRO A 100      17.692  76.451  44.474   1.0   0.0           H
ATOM   1596  HG3 PRO A 100      18.195  77.575  44.343   1.0   0.0           H
ATOM   1597  HD2 PRO A 100      15.512  77.171  44.300   1.0   0.0           H
ATOM   1598  HD3 PRO A 100      15.745  78.363  43.702   1.0   0.0           H
ATOM   1599  N   GLU A 101      16.468  74.054  43.369   1.0   0.0           N
ATOM   1600  CA  GLU A 101      16.361  72.658  43.797   1.0   0.0           C
ATOM   1601  CB  GLU A 101      15.541  72.544  45.088   1.0   0.0           C
ATOM   1602  CG  GLU A 101      16.193  73.172  46.309   1.0   0.0           C
ATOM   1603  CD  GLU A 101      15.474  72.833  47.602   1.0   0.0           C
ATOM   1604  OE1 GLU A 101      14.225  72.886  47.627   1.0   0.0           O
ATOM   1605  OE2 GLU A 101      16.162  72.516  48.596   1.0   0.0           O
ATOM   1606  C   GLU A 101      15.693  71.827  42.711   1.0   0.0           C
ATOM   1607  O   GLU A 101      16.214  70.788  42.296   1.0   0.0           O
ATOM   1608  H   GLU A 101      16.907  74.600  44.083   1.0   0.0           H
ATOM   1609  HA  GLU A 101      17.271  72.274  43.638   1.0   0.0           H
ATOM   1610  HB2 GLU A 101      14.741  72.773  44.533   1.0   0.0           H
ATOM   1611  HB3 GLU A 101      15.028  71.829  44.613   1.0   0.0           H
ATOM   1612  HG2 GLU A 101      17.088  72.898  45.956   1.0   0.0           H
ATOM   1613  HG3 GLU A 101      16.664  73.815  45.705   1.0   0.0           H
ATOM   1614  N   LEU A 102      14.534  72.313  42.268   1.0   0.0           N
ATOM   1615  CA  LEU A 102      13.745  71.689  41.208   1.0   0.0           C
ATOM   1616  CB  LEU A 102      12.435  72.465  41.016   1.0   0.0           C
ATOM   1617  CG  LEU A 102      11.436  72.024  39.938   1.0   0.0           C
ATOM   1618  CD1 LEU A 102      11.129  70.542  40.025   1.0   0.0           C
ATOM   1619  CD2 LEU A 102      10.152  72.823  40.080   1.0   0.0           C
ATOM   1620  C   LEU A 102      14.503  71.628  39.889   1.0   0.0           C
ATOM   1621  O   LEU A 102      14.461  70.622  39.197   1.0   0.0           O
ATOM   1622  H   LEU A 102      14.036  72.353  43.134   1.0   0.0           H
ATOM   1623  HA  LEU A 102      13.851  70.710  41.382   1.0   0.0           H
ATOM   1624  HB2 LEU A 102      12.537  72.501  42.010   1.0   0.0           H
ATOM   1625  HB3 LEU A 102      12.986  73.075  41.586   1.0   0.0           H
ATOM   1626  HG  LEU A 102      12.102  72.091  39.195   1.0   0.0           H
ATOM   1627 HD11 LEU A 102      10.478  70.255  39.322   1.0   0.0           H
ATOM   1628 HD12 LEU A 102      10.626  70.028  40.720   1.0   0.0           H
ATOM   1629 HD13 LEU A 102      11.709  69.751  39.829   1.0   0.0           H
ATOM   1630 HD21 LEU A 102      10.355  73.801  40.023   1.0   0.0           H
ATOM   1631 HD22 LEU A 102       9.592  72.922  40.903   1.0   0.0           H
ATOM   1632 HD23 LEU A 102       9.399  72.875  39.424   1.0   0.0           H
ATOM   1633  N   TYR A 103      15.185  72.712  39.541   1.0   0.0           N
ATOM   1634  CA  TYR A 103      15.921  72.785  38.280   1.0   0.0           C
ATOM   1635  CB  TYR A 103      16.618  74.140  38.145   1.0   0.0           C
ATOM   1636  CG  TYR A 103      17.000  74.489  36.727   1.0   0.0           C
ATOM   1637  CD1 TYR A 103      16.081  74.346  35.675   1.0   0.0           C
ATOM   1638  CE1 TYR A 103      16.415  74.672  34.364   1.0   0.0           C
ATOM   1639  CZ  TYR A 103      17.677  75.159  34.091   1.0   0.0           C
ATOM   1640  OH  TYR A 103      17.988  75.484  32.786   1.0   0.0           O
ATOM   1641  CE2 TYR A 103      18.604  75.317  35.119   1.0   0.0           C
ATOM   1642  CD2 TYR A 103      18.262  74.986  36.427   1.0   0.0           C
ATOM   1643  C   TYR A 103      16.940  71.662  38.161   1.0   0.0           C
ATOM   1644  O   TYR A 103      16.953  70.945  37.164   1.0   0.0           O
ATOM   1645  H   TYR A 103      14.515  73.450  39.619   1.0   0.0           H
ATOM   1646  HA  TYR A 103      15.291  72.354  37.634   1.0   0.0           H
ATOM   1647  HB2 TYR A 103      15.938  74.517  38.774   1.0   0.0           H
ATOM   1648  HB3 TYR A 103      16.801  74.097  39.127   1.0   0.0           H
ATOM   1649  HD1 TYR A 103      15.163  74.001  35.873   1.0   0.0           H
ATOM   1650  HE1 TYR A 103      15.748  74.555  33.628   1.0   0.0           H
ATOM   1651  HH  TYR A 103      17.323  75.371  32.048   1.0   0.0           H
ATOM   1652  HE2 TYR A 103      19.518  75.669  34.917   1.0   0.0           H
ATOM   1653  HD2 TYR A 103      18.932  75.107  37.159   1.0   0.0           H
ATOM   1654  N   ASN A 104      17.766  71.491  39.189   1.0   0.0           N
ATOM   1655  CA  ASN A 104      18.773  70.420  39.206   1.0   0.0           C
ATOM   1656  CB  ASN A 104      19.632  70.510  40.477   1.0   0.0           C
ATOM   1657  CG  ASN A 104      20.822  71.442  40.313   1.0   0.0           C
ATOM   1658  OD1 ASN A 104      20.936  72.163  39.322   1.0   0.0           O
ATOM   1659  ND2 ASN A 104      21.725  71.412  41.279   1.0   0.0           N
ATOM   1660  C   ASN A 104      18.198  69.007  39.072   1.0   0.0           C
ATOM   1661  O   ASN A 104      18.832  68.127  38.492   1.0   0.0           O
ATOM   1662  H   ASN A 104      17.207  71.432  38.362   1.0   0.0           H
ATOM   1663  HA  ASN A 104      19.088  70.446  38.257   1.0   0.0           H
ATOM   1664  HB2 ASN A 104      18.800  70.594  41.025   1.0   0.0           H
ATOM   1665  HB3 ASN A 104      19.219  69.660  40.805   1.0   0.0           H
ATOM   1666 HD21 ASN A 104      21.632  70.826  42.084   1.0   0.0           H
ATOM   1667 HD22 ASN A 104      22.532  71.998  41.201   1.0   0.0           H
ATOM   1668  N   LEU A 105      17.003  68.791  39.604   1.0   0.0           N
ATOM   1669  CA  LEU A 105      16.346  67.496  39.479   1.0   0.0           C
ATOM   1670  CB  LEU A 105      15.194  67.400  40.478   1.0   0.0           C
ATOM   1671  CG  LEU A 105      15.607  67.279  41.942   1.0   0.0           C
ATOM   1672  CD1 LEU A 105      14.404  67.494  42.848   1.0   0.0           C
ATOM   1673  CD2 LEU A 105      16.241  65.921  42.203   1.0   0.0           C
ATOM   1674  C   LEU A 105      15.823  67.213  38.067   1.0   0.0           C
ATOM   1675  O   LEU A 105      15.507  66.069  37.744   1.0   0.0           O
ATOM   1676  H   LEU A 105      17.344  68.976  40.526   1.0   0.0           H
ATOM   1677  HA  LEU A 105      17.118  66.867  39.391   1.0   0.0           H
ATOM   1678  HB2 LEU A 105      14.756  68.106  39.922   1.0   0.0           H
ATOM   1679  HB3 LEU A 105      14.645  67.181  39.672   1.0   0.0           H
ATOM   1680  HG  LEU A 105      16.049  68.176  41.941   1.0   0.0           H
ATOM   1681 HD11 LEU A 105      14.675  67.415  43.807   1.0   0.0           H
ATOM   1682 HD12 LEU A 105      13.612  66.896  42.970   1.0   0.0           H
ATOM   1683 HD13 LEU A 105      13.900  68.343  43.008   1.0   0.0           H
ATOM   1684 HD21 LEU A 105      17.032  65.780  41.607   1.0   0.0           H
ATOM   1685 HD22 LEU A 105      15.875  65.013  41.999   1.0   0.0           H
ATOM   1686 HD23 LEU A 105      16.744  65.613  43.010   1.0   0.0           H
ATOM   1687  N   LEU A 106      15.689  68.253  37.246   1.0   0.0           N
ATOM   1688  CA  LEU A 106      15.318  68.089  35.845   1.0   0.0           C
ATOM   1689  CB  LEU A 106      14.658  69.355  35.305   1.0   0.0           C
ATOM   1690  CG  LEU A 106      13.434  69.894  36.043   1.0   0.0           C
ATOM   1691  CD1 LEU A 106      13.023  71.238  35.446   1.0   0.0           C
ATOM   1692  CD2 LEU A 106      12.282  68.903  36.011   1.0   0.0           C
ATOM   1693  C   LEU A 106      16.566  67.795  35.031   1.0   0.0           C
ATOM   1694  O   LEU A 106      16.601  66.844  34.250   1.0   0.0           O
ATOM   1695  H   LEU A 106      14.867  68.447  37.782   1.0   0.0           H
ATOM   1696  HA  LEU A 106      14.958  67.159  35.773   1.0   0.0           H
ATOM   1697  HB2 LEU A 106      15.611  69.614  35.150   1.0   0.0           H
ATOM   1698  HB3 LEU A 106      15.226  69.105  34.521   1.0   0.0           H
ATOM   1699  HG  LEU A 106      14.003  70.189  36.811   1.0   0.0           H
ATOM   1700 HD11 LEU A 106      12.222  71.591  35.929   1.0   0.0           H
ATOM   1701 HD12 LEU A 106      12.627  71.403  34.543   1.0   0.0           H
ATOM   1702 HD13 LEU A 106      13.518  72.105  35.508   1.0   0.0           H
ATOM   1703 HD21 LEU A 106      12.551  68.023  36.402   1.0   0.0           H
ATOM   1704 HD22 LEU A 106      11.855  68.455  35.226   1.0   0.0           H
ATOM   1705 HD23 LEU A 106      11.430  68.906  36.534   1.0   0.0           H
ATOM   1706  N   LEU A 107      17.590  68.618  35.226   1.0   0.0           N
ATOM   1707  CA  LEU A 107      18.858  68.461  34.523   1.0   0.0           C
ATOM   1708  CB  LEU A 107      19.911  69.422  35.088   1.0   0.0           C
ATOM   1709  CG  LEU A 107      19.656  70.932  34.972   1.0   0.0           C
ATOM   1710  CD1 LEU A 107      20.688  71.707  35.789   1.0   0.0           C
ATOM   1711  CD2 LEU A 107      19.639  71.385  33.513   1.0   0.0           C
ATOM   1712  C   LEU A 107      19.382  67.041  34.647   1.0   0.0           C
ATOM   1713  O   LEU A 107      19.862  66.465  33.671   1.0   0.0           O
ATOM   1714  H   LEU A 107      17.245  69.553  35.144   1.0   0.0           H
ATOM   1715  HA  LEU A 107      18.605  68.375  33.560   1.0   0.0           H
ATOM   1716  HB2 LEU A 107      19.951  68.715  35.794   1.0   0.0           H
ATOM   1717  HB3 LEU A 107      20.426  68.565  35.069   1.0   0.0           H
ATOM   1718  HG  LEU A 107      18.908  70.876  35.634   1.0   0.0           H
ATOM   1719 HD11 LEU A 107      20.522  72.690  35.713   1.0   0.0           H
ATOM   1720 HD12 LEU A 107      21.666  71.810  35.608   1.0   0.0           H
ATOM   1721 HD13 LEU A 107      20.780  71.751  36.784   1.0   0.0           H
ATOM   1722 HD21 LEU A 107      18.963  70.878  32.978   1.0   0.0           H
ATOM   1723 HD22 LEU A 107      20.323  71.263  32.794   1.0   0.0           H
ATOM   1724 HD23 LEU A 107      19.332  72.248  33.111   1.0   0.0           H
ATOM   1725  N   LYS A 108      19.259  66.472  35.843   1.0   0.0           N
ATOM   1726  CA  LYS A 108      19.865  65.182  36.158   1.0   0.0           C
ATOM   1727  CB  LYS A 108      19.732  64.905  37.655   1.0   0.0           C
ATOM   1728  CG  LYS A 108      20.676  63.844  38.196   1.0   0.0           C
ATOM   1729  CD  LYS A 108      20.617  63.804  39.721   1.0   0.0           C
ATOM   1730  CE  LYS A 108      21.111  62.474  40.274   1.0   0.0           C
ATOM   1731  NZ  LYS A 108      22.541  62.218  39.934   1.0   0.0           N
ATOM   1732  C   LYS A 108      19.295  64.009  35.369   1.0   0.0           C
ATOM   1733  O   LYS A 108      19.954  62.991  35.243   1.0   0.0           O
ATOM   1734  H   LYS A 108      19.633  67.242  36.361   1.0   0.0           H
ATOM   1735  HA  LYS A 108      20.710  65.214  35.624   1.0   0.0           H
ATOM   1736  HB2 LYS A 108      19.657  65.896  37.766   1.0   0.0           H
ATOM   1737  HB3 LYS A 108      18.850  65.369  37.578   1.0   0.0           H
ATOM   1738  HG2 LYS A 108      20.349  63.160  37.544   1.0   0.0           H
ATOM   1739  HG3 LYS A 108      21.218  63.814  37.356   1.0   0.0           H
ATOM   1740  HD2 LYS A 108      21.023  64.716  39.770   1.0   0.0           H
ATOM   1741  HD3 LYS A 108      19.971  64.567  39.728   1.0   0.0           H
ATOM   1742  HE2 LYS A 108      20.687  62.668  41.158   1.0   0.0           H
ATOM   1743  HE3 LYS A 108      20.172  62.142  40.365   1.0   0.0           H
ATOM   1744  HZ1 LYS A 108      22.865  61.345  40.297   1.0   0.0           H
ATOM   1745  HZ2 LYS A 108      23.369  62.676  40.257   1.0   0.0           H
ATOM   1746  HZ3 LYS A 108      22.965  62.024  39.050   1.0   0.0           H
ATOM   1747  N   ILE A 109      18.094  64.153  34.812   1.0   0.0           N
ATOM   1748  CA  ILE A 109      17.492  63.092  33.992   1.0   0.0           C
ATOM   1749  CB  ILE A 109      16.063  63.441  33.543   1.0   0.0           C
ATOM   1750  CG1 ILE A 109      15.154  63.684  34.747   1.0   0.0           C
ATOM   1751  CD1 ILE A 109      13.944  64.537  34.417   1.0   0.0           C
ATOM   1752  CG2 ILE A 109      15.491  62.333  32.667   1.0   0.0           C
ATOM   1753  C   ILE A 109      18.328  62.866  32.735   1.0   0.0           C
ATOM   1754  O   ILE A 109      18.892  61.786  32.517   1.0   0.0           O
ATOM   1755  H   ILE A 109      17.546  64.301  35.635   1.0   0.0           H
ATOM   1756  HA  ILE A 109      17.745  62.237  34.445   1.0   0.0           H
ATOM   1757  HB  ILE A 109      16.404  64.094  32.866   1.0   0.0           H
ATOM   1758 HG12 ILE A 109      15.244  62.717  34.987   1.0   0.0           H
ATOM   1759 HG13 ILE A 109      15.913  63.421  35.342   1.0   0.0           H
ATOM   1760 HD11 ILE A 109      13.349  64.696  35.205   1.0   0.0           H
ATOM   1761 HD12 ILE A 109      13.131  64.356  33.863   1.0   0.0           H
ATOM   1762 HD13 ILE A 109      13.854  65.504  34.177   1.0   0.0           H
ATOM   1763 HG21 ILE A 109      14.562  62.560  32.375   1.0   0.0           H
ATOM   1764 HG22 ILE A 109      15.226  61.406  32.931   1.0   0.0           H
ATOM   1765 HG23 ILE A 109      15.762  62.070  31.741   1.0   0.0           H
ATOM   1766  N   THR A 110      18.429  63.914  31.929   1.0   0.0           N
ATOM   1767  CA  THR A 110      19.183  63.873  30.701   1.0   0.0           C
ATOM   1768  CB  THR A 110      18.520  64.758  29.638   1.0   0.0           C
ATOM   1769  OG1 THR A 110      18.514  66.124  30.082   1.0   0.0           O
ATOM   1770  CG2 THR A 110      17.091  64.300  29.378   1.0   0.0           C
ATOM   1771  C   THR A 110      20.577  64.400  30.963   1.0   0.0           C
ATOM   1772  O   THR A 110      21.056  65.271  30.232   1.0   0.0           O
ATOM   1773  H   THR A 110      17.508  63.566  31.756   1.0   0.0           H
ATOM   1774  HA  THR A 110      19.471  62.924  30.570   1.0   0.0           H
ATOM   1775  HB  THR A 110      19.234  64.913  28.955   1.0   0.0           H
ATOM   1776  HG1 THR A 110      18.082  66.701  29.389   1.0   0.0           H
ATOM   1777 HG21 THR A 110      16.659  64.877  28.685   1.0   0.0           H
ATOM   1778 HG22 THR A 110      16.300  64.363  29.986   1.0   0.0           H
ATOM   1779 HG23 THR A 110      16.780  63.459  28.934   1.0   0.0           H
ATOM   1780  N   GLU A 111      21.232  63.905  32.015   1.0   0.0           N
ATOM   1781  CA  GLU A 111      22.632  64.256  32.233   1.0   0.0           C
ATOM   1782  CB  GLU A 111      23.088  64.062  33.687   1.0   0.0           C
ATOM   1783  CG  GLU A 111      23.249  62.641  34.201   1.0   0.0           C
ATOM   1784  CD  GLU A 111      23.640  62.629  35.679   1.0   0.0           C
ATOM   1785  OE1 GLU A 111      22.924  62.018  36.509   1.0   0.0           O
ATOM   1786  OE2 GLU A 111      24.663  63.258  36.030   1.0   0.0           O
ATOM   1787  C   GLU A 111      23.433  63.454  31.215   1.0   0.0           C
ATOM   1788  O   GLU A 111      24.404  63.958  30.643   1.0   0.0           O
ATOM   1789  H   GLU A 111      20.706  64.431  32.683   1.0   0.0           H
ATOM   1790  HA  GLU A 111      22.677  65.019  31.589   1.0   0.0           H
ATOM   1791  HB2 GLU A 111      23.669  64.843  33.459   1.0   0.0           H
ATOM   1792  HB3 GLU A 111      22.825  65.026  33.714   1.0   0.0           H
ATOM   1793  HG2 GLU A 111      22.392  62.361  33.768   1.0   0.0           H
ATOM   1794  HG3 GLU A 111      23.350  62.281  33.273   1.0   0.0           H
ATOM   1795  N   ASN A 112      22.993  62.221  30.963   1.0   0.0           N
ATOM   1796  CA  ASN A 112      23.327  61.521  29.722   1.0   0.0           C
ATOM   1797  CB  ASN A 112      23.699  60.067  29.993   1.0   0.0           C
ATOM   1798  CG  ASN A 112      24.160  59.336  28.744   1.0   0.0           C
ATOM   1799  OD1 ASN A 112      23.958  59.792  27.620   1.0   0.0           O
ATOM   1800  ND2 ASN A 112      24.771  58.181  28.943   1.0   0.0           N
ATOM   1801  C   ASN A 112      22.102  61.602  28.813   1.0   0.0           C
ATOM   1802  O   ASN A 112      21.072  60.990  29.104   1.0   0.0           O
ATOM   1803  H   ASN A 112      22.749  63.174  30.785   1.0   0.0           H
ATOM   1804  HA  ASN A 112      23.823  62.200  29.181   1.0   0.0           H
ATOM   1805  HB2 ASN A 112      24.196  60.357  30.811   1.0   0.0           H
ATOM   1806  HB3 ASN A 112      23.183  60.109  30.849   1.0   0.0           H
ATOM   1807 HD21 ASN A 112      24.935  57.810  29.857   1.0   0.0           H
ATOM   1808 HD22 ASN A 112      25.081  57.658  28.149   1.0   0.0           H
ATOM   1809  N   PRO A 113      22.201  62.372  27.715   1.0   0.0           N
ATOM   1810  CA  PRO A 113      21.019  62.692  26.908   1.0   0.0           C
ATOM   1811  CB  PRO A 113      21.491  63.867  26.053   1.0   0.0           C
ATOM   1812  CG  PRO A 113      22.954  63.659  25.921   1.0   0.0           C
ATOM   1813  CD  PRO A 113      23.411  63.035  27.204   1.0   0.0           C
ATOM   1814  C   PRO A 113      20.500  61.553  26.025   1.0   0.0           C
ATOM   1815  O   PRO A 113      19.440  61.705  25.419   1.0   0.0           O
ATOM   1816  HA  PRO A 113      20.223  62.636  27.511   1.0   0.0           H
ATOM   1817  HB2 PRO A 113      20.934  63.751  25.230   1.0   0.0           H
ATOM   1818  HB3 PRO A 113      20.882  64.642  26.221   1.0   0.0           H
ATOM   1819  HG2 PRO A 113      22.953  63.144  25.064   1.0   0.0           H
ATOM   1820  HG3 PRO A 113      23.218  64.354  25.252   1.0   0.0           H
ATOM   1821  HD2 PRO A 113      24.145  62.428  26.901   1.0   0.0           H
ATOM   1822  HD3 PRO A 113      24.132  63.587  27.622   1.0   0.0           H
ATOM   1823  N   GLU A 114      21.227  60.437  25.949   1.0   0.0           N
ATOM   1824  CA  GLU A 114      20.737  59.244  25.238   1.0   0.0           C
ATOM   1825  CB  GLU A 114      21.614  58.925  24.021   1.0   0.0           C
ATOM   1826  CG  GLU A 114      23.118  59.041  24.244   1.0   0.0           C
ATOM   1827  CD  GLU A 114      23.916  58.921  22.948   1.0   0.0           C
ATOM   1828  OE1 GLU A 114      23.478  59.477  21.906   1.0   0.0           O
ATOM   1829  OE2 GLU A 114      24.993  58.279  22.972   1.0   0.0           O
ATOM   1830  C   GLU A 114      20.569  58.023  26.161   1.0   0.0           C
ATOM   1831  O   GLU A 114      20.294  56.925  25.689   1.0   0.0           O
ATOM   1832  H   GLU A 114      21.336  61.230  25.350   1.0   0.0           H
ATOM   1833  HA  GLU A 114      19.757  59.434  25.303   1.0   0.0           H
ATOM   1834  HB2 GLU A 114      20.974  58.174  23.862   1.0   0.0           H
ATOM   1835  HB3 GLU A 114      20.762  59.040  23.511   1.0   0.0           H
ATOM   1836  HG2 GLU A 114      22.956  59.817  24.854   1.0   0.0           H
ATOM   1837  HG3 GLU A 114      22.975  58.843  25.214   1.0   0.0           H
ATOM   1838  N   ALA A 115      20.715  58.238  27.470   1.0   0.0           N
ATOM   1839  CA  ALA A 115      20.386  57.243  28.494   1.0   0.0           C
ATOM   1840  CB  ALA A 115      21.641  56.533  28.973   1.0   0.0           C
ATOM   1841  C   ALA A 115      19.682  57.957  29.658   1.0   0.0           C
ATOM   1842  O   ALA A 115      20.238  58.084  30.769   1.0   0.0           O
ATOM   1843  H   ALA A 115      19.889  58.705  27.155   1.0   0.0           H
ATOM   1844  HA  ALA A 115      19.527  56.847  28.171   1.0   0.0           H
ATOM   1845  HB1 ALA A 115      22.099  56.068  28.215   1.0   0.0           H
ATOM   1846  HB2 ALA A 115      21.720  55.728  29.561   1.0   0.0           H
ATOM   1847  HB3 ALA A 115      22.500  56.929  29.296   1.0   0.0           H
ATOM   1848  N   PRO A 116      18.460  58.447  29.404   1.0   0.0           N
ATOM   1849  CA  PRO A 116      17.761  59.219  30.412   1.0   0.0           C
ATOM   1850  CB  PRO A 116      16.570  59.800  29.641   1.0   0.0           C
ATOM   1851  CG  PRO A 116      16.304  58.808  28.570   1.0   0.0           C
ATOM   1852  CD  PRO A 116      17.642  58.245  28.197   1.0   0.0           C
ATOM   1853  C   PRO A 116      17.299  58.324  31.547   1.0   0.0           C
ATOM   1854  O   PRO A 116      16.651  57.309  31.300   1.0   0.0           O
ATOM   1855  HA  PRO A 116      18.418  59.675  31.012   1.0   0.0           H
ATOM   1856  HB2 PRO A 116      15.890  59.889  30.369   1.0   0.0           H
ATOM   1857  HB3 PRO A 116      16.610  60.796  29.717   1.0   0.0           H
ATOM   1858  HG2 PRO A 116      15.632  58.236  29.040   1.0   0.0           H
ATOM   1859  HG3 PRO A 116      15.458  59.124  28.141   1.0   0.0           H
ATOM   1860  HD2 PRO A 116      17.415  57.299  27.966   1.0   0.0           H
ATOM   1861  HD3 PRO A 116      17.791  58.361  27.215   1.0   0.0           H
ATOM   1862  N   LYS A 117      17.613  58.706  32.779   1.0   0.0           N
ATOM   1863  CA  LYS A 117      17.254  57.902  33.942   1.0   0.0           C
ATOM   1864  CB  LYS A 117      18.476  57.696  34.846   1.0   0.0           C
ATOM   1865  CG  LYS A 117      18.886  58.910  35.676   1.0   0.0           C
ATOM   1866  CD  LYS A 117      20.343  58.839  36.128   1.0   0.0           C
ATOM   1867  CE  LYS A 117      21.281  59.645  35.229   1.0   0.0           C
ATOM   1868  NZ  LYS A 117      21.160  59.419  33.744   1.0   0.0           N
ATOM   1869  C   LYS A 117      16.124  58.555  34.726   1.0   0.0           C
ATOM   1870  O   LYS A 117      15.974  59.763  34.698   1.0   0.0           O
ATOM   1871  H   LYS A 117      18.355  58.277  32.264   1.0   0.0           H
ATOM   1872  HA  LYS A 117      16.647  57.207  33.556   1.0   0.0           H
ATOM   1873  HB2 LYS A 117      18.078  56.801  35.049   1.0   0.0           H
ATOM   1874  HB3 LYS A 117      18.643  56.895  34.271   1.0   0.0           H
ATOM   1875  HG2 LYS A 117      18.431  59.510  35.018   1.0   0.0           H
ATOM   1876  HG3 LYS A 117      17.922  59.115  35.844   1.0   0.0           H
ATOM   1877  HD2 LYS A 117      20.042  58.979  37.071   1.0   0.0           H
ATOM   1878  HD3 LYS A 117      20.116  58.029  36.669   1.0   0.0           H
ATOM   1879  HE2 LYS A 117      21.080  60.388  35.867   1.0   0.0           H
ATOM   1880  HE3 LYS A 117      21.769  59.785  36.091   1.0   0.0           H
ATOM   1881  HZ1 LYS A 117      21.773  59.946  33.156   1.0   0.0           H
ATOM   1882  HZ2 LYS A 117      20.484  59.616  33.034   1.0   0.0           H
ATOM   1883  HZ3 LYS A 117      21.361  58.676  33.106   1.0   0.0           H
ATOM   1884  N   PRO A 118      15.308  57.750  35.413   1.0   0.0           N
ATOM   1885  CA  PRO A 118      14.380  58.336  36.372   1.0   0.0           C
ATOM   1886  CB  PRO A 118      13.575  57.136  36.872   1.0   0.0           C
ATOM   1887  CG  PRO A 118      14.400  55.941  36.559   1.0   0.0           C
ATOM   1888  CD  PRO A 118      15.157  56.287  35.317   1.0   0.0           C
ATOM   1889  C   PRO A 118      15.127  59.011  37.526   1.0   0.0           C
ATOM   1890  O   PRO A 118      16.179  58.522  37.957   1.0   0.0           O
ATOM   1891  HA  PRO A 118      14.051  59.215  36.028   1.0   0.0           H
ATOM   1892  HB2 PRO A 118      13.439  57.409  37.824   1.0   0.0           H
ATOM   1893  HB3 PRO A 118      12.620  57.430  36.830   1.0   0.0           H
ATOM   1894  HG2 PRO A 118      14.885  55.842  37.428   1.0   0.0           H
ATOM   1895  HG3 PRO A 118      13.898  55.174  36.959   1.0   0.0           H
ATOM   1896  HD2 PRO A 118      16.004  55.764  35.417   1.0   0.0           H
ATOM   1897  HD3 PRO A 118      14.926  55.668  34.567   1.0   0.0           H
ATOM   1898  N   VAL A 119      14.587  60.129  38.002   1.0   0.0           N
ATOM   1899  CA  VAL A 119      15.282  60.988  38.954   1.0   0.0           C
ATOM   1900  CB  VAL A 119      15.895  62.204  38.238   1.0   0.0           C
ATOM   1901  CG1 VAL A 119      16.224  63.330  39.216   1.0   0.0           C
ATOM   1902  CG2 VAL A 119      17.143  61.780  37.485   1.0   0.0           C
ATOM   1903  C   VAL A 119      14.343  61.483  40.041   1.0   0.0           C
ATOM   1904  O   VAL A 119      13.378  62.210  39.750   1.0   0.0           O
ATOM   1905  H   VAL A 119      15.205  59.803  37.287   1.0   0.0           H
ATOM   1906  HA  VAL A 119      15.768  60.351  39.552   1.0   0.0           H
ATOM   1907  HB  VAL A 119      15.023  62.492  37.843   1.0   0.0           H
ATOM   1908 HG11 VAL A 119      16.622  64.120  38.751   1.0   0.0           H
ATOM   1909 HG12 VAL A 119      16.923  63.383  39.929   1.0   0.0           H
ATOM   1910 HG13 VAL A 119      15.618  63.929  39.739   1.0   0.0           H
ATOM   1911 HG21 VAL A 119      17.541  62.570  37.020   1.0   0.0           H
ATOM   1912 HG22 VAL A 119      17.188  61.198  36.673   1.0   0.0           H
ATOM   1913 HG23 VAL A 119      18.015  61.492  37.880   1.0   0.0           H
ATOM   1914  N   CYS A 120      14.649  61.105  41.290   1.0   0.0           N
ATOM   1915  CA  CYS A 120      13.848  61.489  42.456   1.0   0.0           C
ATOM   1916  CB  CYS A 120      13.277  60.251  43.126   1.0   0.0           C
ATOM   1917  SG  CYS A 120      12.122  59.356  42.079   1.0   0.0           S
ATOM   1918  C   CYS A 120      14.651  62.284  43.480   1.0   0.0           C
ATOM   1919  O   CYS A 120      15.817  61.974  43.744   1.0   0.0           O
ATOM   1920  H   CYS A 120      14.122  60.584  40.618   1.0   0.0           H
ATOM   1921  HA  CYS A 120      13.325  62.278  42.135   1.0   0.0           H
ATOM   1922  HB2 CYS A 120      14.207  60.142  43.478   1.0   0.0           H
ATOM   1923  HB3 CYS A 120      13.713  60.650  43.933   1.0   0.0           H
ATOM   1924  HG  CYS A 120      11.746  58.541  42.520   1.0   0.0           H
ATOM   1925  N   GLY A 121      14.012  63.291  44.075   1.0   0.0           N
ATOM   1926  CA  GLY A 121      14.642  64.090  45.127   1.0   0.0           C
ATOM   1927  C   GLY A 121      13.677  65.035  45.815   1.0   0.0           C
ATOM   1928  O   GLY A 121      12.468  64.999  45.564   1.0   0.0           O
ATOM   1929  H   GLY A 121      14.649  62.668  43.621   1.0   0.0           H
ATOM   1930  HA2 GLY A 121      15.111  63.267  45.446   1.0   0.0           H
ATOM   1931  HA3 GLY A 121      15.532  63.877  44.723   1.0   0.0           H
ATOM   1932  N   TYR A 122      14.216  65.893  46.677   1.0   0.0           N
ATOM   1933  CA  TYR A 122      13.401  66.844  47.445   1.0   0.0           C
ATOM   1934  CB  TYR A 122      13.737  66.766  48.943   1.0   0.0           C
ATOM   1935  CG  TYR A 122      13.144  65.556  49.621   1.0   0.0           C
ATOM   1936  CD1 TYR A 122      11.843  65.582  50.108   1.0   0.0           C
ATOM   1937  CE1 TYR A 122      11.282  64.470  50.713   1.0   0.0           C
ATOM   1938  CZ  TYR A 122      12.028  63.308  50.830   1.0   0.0           C
ATOM   1939  OH  TYR A 122      11.465  62.207  51.432   1.0   0.0           O
ATOM   1940  CE2 TYR A 122      13.325  63.260  50.354   1.0   0.0           C
ATOM   1941  CD2 TYR A 122      13.873  64.378  49.747   1.0   0.0           C
ATOM   1942  C   TYR A 122      13.538  68.282  46.946   1.0   0.0           C
ATOM   1943  O   TYR A 122      14.607  68.707  46.506   1.0   0.0           O
ATOM   1944  H   TYR A 122      14.126  64.952  47.004   1.0   0.0           H
ATOM   1945  HA  TYR A 122      12.512  66.728  47.002   1.0   0.0           H
ATOM   1946  HB2 TYR A 122      14.682  67.011  48.728   1.0   0.0           H
ATOM   1947  HB3 TYR A 122      13.965  67.740  48.922   1.0   0.0           H
ATOM   1948  HD1 TYR A 122      11.302  66.419  50.020   1.0   0.0           H
ATOM   1949  HE1 TYR A 122      10.346  64.505  51.063   1.0   0.0           H
ATOM   1950  HH  TYR A 122      10.527  62.242  51.776   1.0   0.0           H
ATOM   1951  HE2 TYR A 122      13.864  62.423  50.447   1.0   0.0           H
ATOM   1952  HD2 TYR A 122      14.808  64.338  49.394   1.0   0.0           H
ATOM   1953  N   TYR A 123      12.431  69.013  47.017   1.0   0.0           N
ATOM   1954  CA  TYR A 123      12.407  70.434  46.702   1.0   0.0           C
ATOM   1955  CB  TYR A 123      12.029  70.678  45.232   1.0   0.0           C
ATOM   1956  CG  TYR A 123      10.561  70.446  44.861   1.0   0.0           C
ATOM   1957  CD1 TYR A 123       9.974  69.187  44.981   1.0   0.0           C
ATOM   1958  CE1 TYR A 123       8.650  68.976  44.630   1.0   0.0           C
ATOM   1959  CZ  TYR A 123       7.889  70.025  44.138   1.0   0.0           C
ATOM   1960  OH  TYR A 123       6.569  69.811  43.801   1.0   0.0           O
ATOM   1961  CE2 TYR A 123       8.449  71.284  43.994   1.0   0.0           C
ATOM   1962  CD2 TYR A 123       9.777  71.487  44.348   1.0   0.0           C
ATOM   1963  C   TYR A 123      11.429  71.129  47.638   1.0   0.0           C
ATOM   1964  O   TYR A 123      10.619  70.471  48.299   1.0   0.0           O
ATOM   1965  H   TYR A 123      13.074  68.556  46.402   1.0   0.0           H
ATOM   1966  HA  TYR A 123      13.190  70.777  47.221   1.0   0.0           H
ATOM   1967  HB2 TYR A 123      12.641  71.465  45.314   1.0   0.0           H
ATOM   1968  HB3 TYR A 123      13.020  70.637  45.107   1.0   0.0           H
ATOM   1969  HD1 TYR A 123      10.518  68.422  45.327   1.0   0.0           H
ATOM   1970  HE1 TYR A 123       8.244  68.068  44.732   1.0   0.0           H
ATOM   1971  HH  TYR A 123       6.165  68.902  43.905   1.0   0.0           H
ATOM   1972  HE2 TYR A 123       7.901  72.041  43.638   1.0   0.0           H
ATOM   1973  HD2 TYR A 123      10.181  72.395  44.234   1.0   0.0           H
ATOM   1974  N   HIS A 124      11.519  72.453  47.704   1.0   0.0           N
ATOM   1975  CA  HIS A 124      10.595  73.229  48.520   1.0   0.0           C
ATOM   1976  CB  HIS A 124      11.361  74.185  49.424   1.0   0.0           C
ATOM   1977  CG  HIS A 124      12.237  73.489  50.417   1.0   0.0           C
ATOM   1978  ND1 HIS A 124      13.614  73.515  50.343   1.0   0.0           N
ATOM   1979  CE1 HIS A 124      14.119  72.810  51.341   1.0   0.0           C
ATOM   1980  NE2 HIS A 124      13.120  72.322  52.056   1.0   0.0           N
ATOM   1981  CD2 HIS A 124      11.931  72.728  51.497   1.0   0.0           C
ATOM   1982  C   HIS A 124       9.596  73.958  47.625   1.0   0.0           C
ATOM   1983  O   HIS A 124       9.830  74.088  46.425   1.0   0.0           O
ATOM   1984  H   HIS A 124      12.173  71.975  48.290   1.0   0.0           H
ATOM   1985  HA  HIS A 124       9.903  72.540  48.736   1.0   0.0           H
ATOM   1986  HB2 HIS A 124      11.578  74.743  48.623   1.0   0.0           H
ATOM   1987  HB3 HIS A 124      10.709  74.903  49.181   1.0   0.0           H
ATOM   1988  HE1 HIS A 124      15.092  72.670  51.524   1.0   0.0           H
ATOM   1989  HE2 HIS A 124      13.214  71.750  52.871   1.0   0.0           H
ATOM   1990  HD2 HIS A 124      11.013  72.504  51.825   1.0   0.0           H
ATOM   1991  N   TRP A 125       8.483  74.410  48.211   1.0   0.0           N
ATOM   1992  CA  TRP A 125       7.321  74.871  47.442   1.0   0.0           C
ATOM   1993  CB  TRP A 125       6.251  73.776  47.464   1.0   0.0           C
ATOM   1994  CG  TRP A 125       5.336  73.838  46.300   1.0   0.0           C
ATOM   1995  CD1 TRP A 125       5.690  73.913  44.972   1.0   0.0           C
ATOM   1996  NE1 TRP A 125       4.565  73.944  44.184   1.0   0.0           N
ATOM   1997  CE2 TRP A 125       3.456  73.884  44.992   1.0   0.0           C
ATOM   1998  CD2 TRP A 125       3.906  73.813  46.333   1.0   0.0           C
ATOM   1999  CE3 TRP A 125       2.956  73.737  47.363   1.0   0.0           C
ATOM   2000  CZ3 TRP A 125       1.602  73.738  47.028   1.0   0.0           C
ATOM   2001  CH2 TRP A 125       1.188  73.811  45.685   1.0   0.0           C
ATOM   2002  CZ2 TRP A 125       2.096  73.883  44.657   1.0   0.0           C
ATOM   2003  C   TRP A 125       6.728  76.197  47.933   1.0   0.0           C
ATOM   2004  O   TRP A 125       6.128  76.950  47.157   1.0   0.0           O
ATOM   2005  H   TRP A 125       8.870  73.545  47.891   1.0   0.0           H
ATOM   2006  HA  TRP A 125       7.746  75.298  46.644   1.0   0.0           H
ATOM   2007  HB2 TRP A 125       6.947  73.102  47.711   1.0   0.0           H
ATOM   2008  HB3 TRP A 125       6.399  73.653  48.445   1.0   0.0           H
ATOM   2009  HD1 TRP A 125       6.629  73.941  44.630   1.0   0.0           H
ATOM   2010  HE1 TRP A 125       4.555  74.001  43.186   1.0   0.0           H
ATOM   2011  HE3 TRP A 125       3.244  73.683  48.319   1.0   0.0           H
ATOM   2012  HZ3 TRP A 125       0.915  73.686  47.753   1.0   0.0           H
ATOM   2013  HH2 TRP A 125       0.210  73.810  45.474   1.0   0.0           H
ATOM   2014  HZ2 TRP A 125       1.797  73.933  43.704   1.0   0.0           H
ATOM   2015  N   GLU A 126       7.442  77.275  51.019   1.0   0.0           N
ATOM   2016  CA  GLU A 126       8.633  76.560  51.487   1.0   0.0           C
ATOM   2017  CB  GLU A 126       9.352  77.377  52.572   1.0   0.0           C
ATOM   2018  CG  GLU A 126      10.150  78.561  52.037   1.0   0.0           C
ATOM   2019  CD  GLU A 126      11.361  78.143  51.208   1.0   0.0           C
ATOM   2020  OE1 GLU A 126      12.470  78.065  51.781   1.0   0.0           O
ATOM   2021  OE2 GLU A 126      11.208  77.884  49.986   1.0   0.0           O
ATOM   2022  C   GLU A 126       8.351  75.135  51.992   1.0   0.0           C
ATOM   2023  O   GLU A 126       9.147  74.558  52.739   1.0   0.0           O
ATOM   2024  H   GLU A 126       7.625  78.202  50.691   1.0   0.0           H
ATOM   2025  HA  GLU A 126       8.998  76.227  50.618   1.0   0.0           H
ATOM   2026  HB2 GLU A 126       8.517  77.298  53.116   1.0   0.0           H
ATOM   2027  HB3 GLU A 126       9.213  76.587  53.169   1.0   0.0           H
ATOM   2028  HG2 GLU A 126       9.287  79.002  51.790   1.0   0.0           H
ATOM   2029  HG3 GLU A 126       9.698  79.135  52.720   1.0   0.0           H
ATOM   2030  N   ILE A 127       7.228  74.570  51.554   1.0   0.0           N
ATOM   2031  CA  ILE A 127       6.860  73.187  51.865   1.0   0.0           C
ATOM   2032  CB  ILE A 127       5.453  72.828  51.317   1.0   0.0           C
ATOM   2033  CG1 ILE A 127       4.360  73.592  52.070   1.0   0.0           C
ATOM   2034  CD1 ILE A 127       2.981  73.421  51.466   1.0   0.0           C
ATOM   2035  CG2 ILE A 127       5.182  71.324  51.413   1.0   0.0           C
ATOM   2036  C   ILE A 127       7.841  72.229  51.207   1.0   0.0           C
ATOM   2037  O   ILE A 127       8.037  72.301  49.995   1.0   0.0           O
ATOM   2038  H   ILE A 127       6.583  75.200  51.987   1.0   0.0           H
ATOM   2039  HA  ILE A 127       7.135  73.017  52.811   1.0   0.0           H
ATOM   2040  HB  ILE A 127       5.762  72.934  50.372   1.0   0.0           H
ATOM   2041 HG12 ILE A 127       4.762  73.272  52.928   1.0   0.0           H
ATOM   2042 HG13 ILE A 127       5.081  74.145  52.488   1.0   0.0           H
ATOM   2043 HD11 ILE A 127       2.267  73.920  51.958   1.0   0.0           H
ATOM   2044 HD12 ILE A 127       2.362  72.637  51.414   1.0   0.0           H
ATOM   2045 HD13 ILE A 127       2.579  73.741  50.608   1.0   0.0           H
ATOM   2046 HG21 ILE A 127       4.275  71.093  51.060   1.0   0.0           H
ATOM   2047 HG22 ILE A 127       5.035  70.749  52.218   1.0   0.0           H
ATOM   2048 HG23 ILE A 127       5.592  70.565  50.907   1.0   0.0           H
ATOM   2049  N   PRO A 128       8.426  71.303  51.989   1.0   0.0           N
ATOM   2050  CA  PRO A 128       9.262  70.295  51.366   1.0   0.0           C
ATOM   2051  CB  PRO A 128      10.031  69.705  52.550   1.0   0.0           C
ATOM   2052  CG  PRO A 128       9.086  69.823  53.696   1.0   0.0           C
ATOM   2053  CD  PRO A 128       8.190  71.005  53.416   1.0   0.0           C
ATOM   2054  C   PRO A 128       8.400  69.226  50.690   1.0   0.0           C
ATOM   2055  O   PRO A 128       7.529  68.621  51.344   1.0   0.0           O
ATOM   2056  HA  PRO A 128       9.612  70.616  50.486   1.0   0.0           H
ATOM   2057  HB2 PRO A 128      10.235  68.785  52.214   1.0   0.0           H
ATOM   2058  HB3 PRO A 128      11.007  69.854  52.394   1.0   0.0           H
ATOM   2059  HG2 PRO A 128       8.757  68.879  53.686   1.0   0.0           H
ATOM   2060  HG3 PRO A 128       9.606  69.363  54.415   1.0   0.0           H
ATOM   2061  HD2 PRO A 128       7.290  70.643  53.657   1.0   0.0           H
ATOM   2062  HD3 PRO A 128       8.093  71.546  54.251   1.0   0.0           H
ATOM   2063  N   LYS A 129       8.657  68.996  49.402   1.0   0.0           N
ATOM   2064  CA  LYS A 129       7.983  67.948  48.658   1.0   0.0           C
ATOM   2065  CB  LYS A 129       7.099  68.570  47.581   1.0   0.0           C
ATOM   2066  CG  LYS A 129       5.835  69.206  48.122   1.0   0.0           C
ATOM   2067  CD  LYS A 129       4.946  69.749  47.009   1.0   0.0           C
ATOM   2068  CE  LYS A 129       3.545  70.064  47.524   1.0   0.0           C
ATOM   2069  NZ  LYS A 129       2.599  70.520  46.475   1.0   0.0           N
ATOM   2070  C   LYS A 129       8.995  66.979  48.036   1.0   0.0           C
ATOM   2071  O   LYS A 129      10.099  67.367  47.652   1.0   0.0           O
ATOM   2072  H   LYS A 129       7.997  69.628  49.808   1.0   0.0           H
ATOM   2073  HA  LYS A 129       7.733  67.290  49.369   1.0   0.0           H
ATOM   2074  HB2 LYS A 129       7.926  68.951  47.169   1.0   0.0           H
ATOM   2075  HB3 LYS A 129       7.601  68.021  46.913   1.0   0.0           H
ATOM   2076  HG2 LYS A 129       5.703  68.431  48.740   1.0   0.0           H
ATOM   2077  HG3 LYS A 129       6.255  69.288  49.026   1.0   0.0           H
ATOM   2078  HD2 LYS A 129       5.667  70.372  46.705   1.0   0.0           H
ATOM   2079  HD3 LYS A 129       5.547  69.376  46.302   1.0   0.0           H
ATOM   2080  HE2 LYS A 129       3.594  69.229  48.071   1.0   0.0           H
ATOM   2081  HE3 LYS A 129       3.942  70.127  48.439   1.0   0.0           H
ATOM   2082  HZ1 LYS A 129       1.681  70.726  46.813   1.0   0.0           H
ATOM   2083  HZ2 LYS A 129       2.173  70.052  45.701   1.0   0.0           H
ATOM   2084  HZ3 LYS A 129       2.550  71.355  45.928   1.0   0.0           H
ATOM   2085  N   TYR A 130       8.608  65.711  47.936   1.0   0.0           N
ATOM   2086  CA  TYR A 130       9.409  64.712  47.224   1.0   0.0           C
ATOM   2087  CB  TYR A 130       9.300  63.359  47.911   1.0   0.0           C
ATOM   2088  CG  TYR A 130      10.235  62.283  47.385   1.0   0.0           C
ATOM   2089  CD1 TYR A 130      11.616  62.465  47.364   1.0   0.0           C
ATOM   2090  CE1 TYR A 130      12.465  61.469  46.912   1.0   0.0           C
ATOM   2091  CZ  TYR A 130      11.941  60.274  46.480   1.0   0.0           C
ATOM   2092  OH  TYR A 130      12.759  59.275  46.023   1.0   0.0           O
ATOM   2093  CE2 TYR A 130      10.579  60.068  46.493   1.0   0.0           C
ATOM   2094  CD2 TYR A 130       9.735  61.066  46.951   1.0   0.0           C
ATOM   2095  C   TYR A 130       8.887  64.596  45.802   1.0   0.0           C
ATOM   2096  O   TYR A 130       7.675  64.469  45.583   1.0   0.0           O
ATOM   2097  H   TYR A 130       8.952  65.787  48.872   1.0   0.0           H
ATOM   2098  HA  TYR A 130      10.254  65.175  46.955   1.0   0.0           H
ATOM   2099  HB2 TYR A 130       9.231  63.882  48.760   1.0   0.0           H
ATOM   2100  HB3 TYR A 130       8.427  63.711  48.248   1.0   0.0           H
ATOM   2101  HD1 TYR A 130      12.001  63.331  47.681   1.0   0.0           H
ATOM   2102  HE1 TYR A 130      13.454  61.618  46.900   1.0   0.0           H
ATOM   2103  HH  TYR A 130      13.748  59.425  46.014   1.0   0.0           H
ATOM   2104  HE2 TYR A 130      10.200  59.200  46.172   1.0   0.0           H
ATOM   2105  HD2 TYR A 130       8.748  60.905  46.969   1.0   0.0           H
ATOM   2106  N   LEU A 131       9.799  64.608  44.840   1.0   0.0           N
ATOM   2107  CA  LEU A 131       9.424  64.672  43.438   1.0   0.0           C
ATOM   2108  CB  LEU A 131       9.792  66.046  42.904   1.0   0.0           C
ATOM   2109  CG  LEU A 131       9.780  66.274  41.396   1.0   0.0           C
ATOM   2110  CD1 LEU A 131       9.495  67.741  41.119   1.0   0.0           C
ATOM   2111  CD2 LEU A 131      11.091  65.846  40.742   1.0   0.0           C
ATOM   2112  C   LEU A 131      10.122  63.585  42.640   1.0   0.0           C
ATOM   2113  O   LEU A 131      11.304  63.313  42.858   1.0   0.0           O
ATOM   2114  H   LEU A 131       9.339  65.324  45.366   1.0   0.0           H
ATOM   2115  HA  LEU A 131       8.542  64.203  43.388   1.0   0.0           H
ATOM   2116  HB2 LEU A 131       9.277  66.321  43.716   1.0   0.0           H
ATOM   2117  HB3 LEU A 131      10.118  66.107  43.847   1.0   0.0           H
ATOM   2118  HG  LEU A 131       8.864  65.877  41.338   1.0   0.0           H
ATOM   2119 HD11 LEU A 131       9.487  67.890  40.130   1.0   0.0           H
ATOM   2120 HD12 LEU A 131      10.110  68.505  41.311   1.0   0.0           H
ATOM   2121 HD13 LEU A 131       8.627  68.211  41.278   1.0   0.0           H
ATOM   2122 HD21 LEU A 131      11.279  64.881  40.924   1.0   0.0           H
ATOM   2123 HD22 LEU A 131      12.025  66.111  40.982   1.0   0.0           H
ATOM   2124 HD23 LEU A 131      11.300  65.741  39.770   1.0   0.0           H
ATOM   2125  N   CYS A 132       9.384  62.984  41.708   1.0   0.0           N
ATOM   2126  CA  CYS A 132       9.943  61.976  40.807   1.0   0.0           C
ATOM   2127  CB  CYS A 132       9.465  60.579  41.184   1.0   0.0           C
ATOM   2128  SG  CYS A 132      10.263  59.982  42.697   1.0   0.0           S
ATOM   2129  C   CYS A 132       9.599  62.251  39.358   1.0   0.0           C
ATOM   2130  O   CYS A 132       8.558  62.820  39.052   1.0   0.0           O
ATOM   2131  H   CYS A 132       9.611  62.802  42.665   1.0   0.0           H
ATOM   2132  HA  CYS A 132      10.892  62.277  40.712   1.0   0.0           H
ATOM   2133  HB2 CYS A 132       8.558  60.902  40.915   1.0   0.0           H
ATOM   2134  HB3 CYS A 132       9.051  60.585  40.274   1.0   0.0           H
ATOM   2135  HG  CYS A 132       9.949  59.065  42.944   1.0   0.0           H
ATOM   2136  N   HIS A 133      10.504  61.841  38.477   1.0   0.0           N
ATOM   2137  CA  HIS A 133      10.308  61.940  37.040   1.0   0.0           C
ATOM   2138  CB  HIS A 133      11.179  63.037  36.449   1.0   0.0           C
ATOM   2139  CG  HIS A 133      10.579  64.401  36.544   1.0   0.0           C
ATOM   2140  ND1 HIS A 133      11.081  65.374  37.378   1.0   0.0           N
ATOM   2141  CE1 HIS A 133      10.356  66.471  37.251   1.0   0.0           C
ATOM   2142  NE2 HIS A 133       9.396  66.239  36.375   1.0   0.0           N
ATOM   2143  CD2 HIS A 133       9.511  64.950  35.918   1.0   0.0           C
ATOM   2144  C   HIS A 133      10.697  60.636  36.412   1.0   0.0           C
ATOM   2145  O   HIS A 133      11.756  60.093  36.710   1.0   0.0           O
ATOM   2146  H   HIS A 133      10.244  62.711  38.896   1.0   0.0           H
ATOM   2147  HA  HIS A 133       9.327  61.834  36.876   1.0   0.0           H
ATOM   2148  HB2 HIS A 133      11.967  62.617  36.900   1.0   0.0           H
ATOM   2149  HB3 HIS A 133      11.675  62.326  35.951   1.0   0.0           H
ATOM   2150  HE1 HIS A 133      10.509  67.332  37.736   1.0   0.0           H
ATOM   2151  HE2 HIS A 133       8.698  66.896  36.090   1.0   0.0           H
ATOM   2152  HD2 HIS A 133       8.918  64.501  35.249   1.0   0.0           H
ATOM   2153  N   TYR A 134       9.844  60.133  35.543   1.0   0.0           N
ATOM   2154  CA  TYR A 134      10.098  58.873  34.881   1.0   0.0           C
ATOM   2155  CB  TYR A 134       9.108  57.812  35.355   1.0   0.0           C
ATOM   2156  CG  TYR A 134       9.663  56.973  36.452   1.0   0.0           C
ATOM   2157  CD1 TYR A 134       9.717  57.455  37.756   1.0   0.0           C
ATOM   2158  CE1 TYR A 134      10.246  56.692  38.779   1.0   0.0           C
ATOM   2159  CZ  TYR A 134      10.734  55.421  38.491   1.0   0.0           C
ATOM   2160  OH  TYR A 134      11.280  54.639  39.492   1.0   0.0           O
ATOM   2161  CE2 TYR A 134      10.691  54.931  37.190   1.0   0.0           C
ATOM   2162  CD2 TYR A 134      10.160  55.710  36.185   1.0   0.0           C
ATOM   2163  C   TYR A 134      10.023  59.041  33.371   1.0   0.0           C
ATOM   2164  O   TYR A 134       8.938  59.233  32.816   1.0   0.0           O
ATOM   2165  H   TYR A 134       9.893  60.023  36.536   1.0   0.0           H
ATOM   2166  HA  TYR A 134      11.097  58.832  34.865   1.0   0.0           H
ATOM   2167  HB2 TYR A 134       8.371  58.488  35.375   1.0   0.0           H
ATOM   2168  HB3 TYR A 134       8.592  57.885  34.502   1.0   0.0           H
ATOM   2169  HD1 TYR A 134       9.367  58.370  37.956   1.0   0.0           H
ATOM   2170  HE1 TYR A 134      10.278  57.048  39.713   1.0   0.0           H
ATOM   2171  HH  TYR A 134      11.311  54.991  40.427   1.0   0.0           H
ATOM   2172  HE2 TYR A 134      11.044  54.018  36.985   1.0   0.0           H
ATOM   2173  HD2 TYR A 134      10.134  55.357  35.250   1.0   0.0           H
ATOM   2174  N   PRO A 135      11.175  58.956  32.697   1.0   0.0           N
ATOM   2175  CA  PRO A 135      11.181  59.170  31.272   1.0   0.0           C
ATOM   2176  CB  PRO A 135      12.665  59.180  30.943   1.0   0.0           C
ATOM   2177  CG  PRO A 135      13.272  58.260  31.939   1.0   0.0           C
ATOM   2178  CD  PRO A 135      12.473  58.456  33.190   1.0   0.0           C
ATOM   2179  C   PRO A 135      10.484  58.039  30.545   1.0   0.0           C
ATOM   2180  O   PRO A 135      10.504  56.903  31.014   1.0   0.0           O
ATOM   2181  HA  PRO A 135      10.483  59.833  31.002   1.0   0.0           H
ATOM   2182  HB2 PRO A 135      12.654  58.888  29.987   1.0   0.0           H
ATOM   2183  HB3 PRO A 135      12.883  60.070  30.543   1.0   0.0           H
ATOM   2184  HG2 PRO A 135      13.199  57.393  31.446   1.0   0.0           H
ATOM   2185  HG3 PRO A 135      14.227  58.151  31.664   1.0   0.0           H
ATOM   2186  HD2 PRO A 135      12.466  57.538  33.588   1.0   0.0           H
ATOM   2187  HD3 PRO A 135      13.075  58.702  33.950   1.0   0.0           H
ATOM   2188  N   THR A 136       9.850  58.354  29.426   1.0   0.0           N
ATOM   2189  CA  THR A 136       9.397  57.335  28.495   1.0   0.0           C
ATOM   2190  CB  THR A 136       8.186  57.809  27.701   1.0   0.0           C
ATOM   2191  OG1 THR A 136       8.571  58.947  26.943   1.0   0.0           O
ATOM   2192  CG2 THR A 136       7.075  58.197  28.631   1.0   0.0           C
ATOM   2193  C   THR A 136      10.531  57.109  27.517   1.0   0.0           C
ATOM   2194  O   THR A 136      11.553  57.783  27.599   1.0   0.0           O
ATOM   2195  H   THR A 136       9.101  58.503  30.072   1.0   0.0           H
ATOM   2196  HA  THR A 136       9.460  56.443  28.942   1.0   0.0           H
ATOM   2197  HB  THR A 136       8.094  57.202  26.911   1.0   0.0           H
ATOM   2198  HG1 THR A 136       7.776  59.258  26.422   1.0   0.0           H
ATOM   2199 HG21 THR A 136       6.280  58.508  28.110   1.0   0.0           H
ATOM   2200 HG22 THR A 136       7.049  58.982  29.250   1.0   0.0           H
ATOM   2201 HG23 THR A 136       6.531  57.591  29.211   1.0   0.0           H
ATOM   2202  N   THR A 137      10.343  56.162  26.596   1.0   0.0           N
ATOM   2203  CA  THR A 137      11.293  55.950  25.492   1.0   0.0           C
ATOM   2204  CB  THR A 137      11.550  54.440  25.218   1.0   0.0           C
ATOM   2205  OG1 THR A 137      10.363  53.810  24.716   1.0   0.0           O
ATOM   2206  CG2 THR A 137      12.019  53.717  26.494   1.0   0.0           C
ATOM   2207  C   THR A 137      10.826  56.634  24.210   1.0   0.0           C
ATOM   2208  O   THR A 137      11.121  56.172  23.121   1.0   0.0           O
ATOM   2209  H   THR A 137      10.649  55.714  27.436   1.0   0.0           H
ATOM   2210  HA  THR A 137      11.966  56.663  25.687   1.0   0.0           H
ATOM   2211  HB  THR A 137      11.997  54.445  24.323   1.0   0.0           H
ATOM   2212  HG1 THR A 137      10.528  52.840  24.540   1.0   0.0           H
ATOM   2213 HG21 THR A 137      12.184  52.747  26.318   1.0   0.0           H
ATOM   2214 HG22 THR A 137      11.512  53.502  27.329   1.0   0.0           H
ATOM   2215 HG23 THR A 137      12.883  53.797  26.992   1.0   0.0           H
ATOM   2216  N   ILE A 138      10.080  57.719  24.359   1.0   0.0           N
ATOM   2217  CA  ILE A 138       9.472  58.432  23.250   1.0   0.0           C
ATOM   2218  CB  ILE A 138       7.958  58.626  23.479   1.0   0.0           C
ATOM   2219  CG1 ILE A 138       7.256  57.266  23.381   1.0   0.0           C
ATOM   2220  CD1 ILE A 138       5.761  57.325  23.132   1.0   0.0           C
ATOM   2221  CG2 ILE A 138       7.361  59.646  22.511   1.0   0.0           C
ATOM   2222  C   ILE A 138      10.125  59.791  23.100   1.0   0.0           C
ATOM   2223  O   ILE A 138      10.397  60.467  24.074   1.0   0.0           O
ATOM   2224  H   ILE A 138       9.649  56.822  24.458   1.0   0.0           H
ATOM   2225  HA  ILE A 138       9.906  58.052  22.433   1.0   0.0           H
ATOM   2226  HB  ILE A 138       8.080  59.175  24.306   1.0   0.0           H
ATOM   2227 HG12 ILE A 138       8.049  56.977  22.844   1.0   0.0           H
ATOM   2228 HG13 ILE A 138       8.084  56.912  23.815   1.0   0.0           H
ATOM   2229 HD11 ILE A 138       5.303  56.438  23.068   1.0   0.0           H
ATOM   2230 HD12 ILE A 138       5.186  57.596  22.360   1.0   0.0           H
ATOM   2231 HD13 ILE A 138       4.968  57.614  23.669   1.0   0.0           H
ATOM   2232 HG21 ILE A 138       6.380  59.772  22.659   1.0   0.0           H
ATOM   2233 HG22 ILE A 138       7.212  59.594  21.523   1.0   0.0           H
ATOM   2234 HG23 ILE A 138       7.486  60.636  22.453   1.0   0.0           H
ATOM   2235  N   LYS A 139      10.364  60.178  21.854   1.0   0.0           N
ATOM   2236  CA  LYS A 139      10.856  61.501  21.536   1.0   0.0           C
ATOM   2237  CB  LYS A 139      12.167  61.410  20.771   1.0   0.0           C
ATOM   2238  CG  LYS A 139      13.352  61.061  21.643   1.0   0.0           C
ATOM   2239  CD  LYS A 139      14.637  61.192  20.862   1.0   0.0           C
ATOM   2240  CE  LYS A 139      15.827  60.881  21.741   1.0   0.0           C
ATOM   2241  NZ  LYS A 139      17.091  61.260  21.064   1.0   0.0           N
ATOM   2242  C   LYS A 139       9.817  62.252  20.716   1.0   0.0           C
ATOM   2243  O   LYS A 139       8.999  61.648  20.019   1.0   0.0           O
ATOM   2244  H   LYS A 139      11.047  59.685  22.393   1.0   0.0           H
ATOM   2245  HA  LYS A 139      10.661  62.032  22.361   1.0   0.0           H
ATOM   2246  HB2 LYS A 139      11.684  60.903  20.057   1.0   0.0           H
ATOM   2247  HB3 LYS A 139      11.732  61.928  20.035   1.0   0.0           H
ATOM   2248  HG2 LYS A 139      13.031  61.635  22.396   1.0   0.0           H
ATOM   2249  HG3 LYS A 139      12.805  60.542  22.300   1.0   0.0           H
ATOM   2250  HD2 LYS A 139      14.280  60.658  20.096   1.0   0.0           H
ATOM   2251  HD3 LYS A 139      14.224  61.768  20.157   1.0   0.0           H
ATOM   2252  HE2 LYS A 139      15.415  61.310  22.545   1.0   0.0           H
ATOM   2253  HE3 LYS A 139      15.339  60.206  22.294   1.0   0.0           H
ATOM   2254  HZ1 LYS A 139      17.878  61.054  21.645   1.0   0.0           H
ATOM   2255  HZ2 LYS A 139      17.416  62.182  20.856   1.0   0.0           H
ATOM   2256  HZ3 LYS A 139      17.503  60.831  20.260   1.0   0.0           H
ATOM   2257  N   VAL A 140       9.833  63.574  20.830   1.0   0.0           N
ATOM   2258  CA  VAL A 140       8.951  64.421  20.044   1.0   0.0           C
ATOM   2259  CB  VAL A 140       7.693  64.858  20.822   1.0   0.0           C
ATOM   2260  CG1 VAL A 140       6.890  63.647  21.260   1.0   0.0           C
ATOM   2261  CG2 VAL A 140       8.054  65.733  22.014   1.0   0.0           C
ATOM   2262  C   VAL A 140       9.729  65.633  19.581   1.0   0.0           C
ATOM   2263  O   VAL A 140      10.613  66.127  20.290   1.0   0.0           O
ATOM   2264  H   VAL A 140       9.319  62.773  21.136   1.0   0.0           H
ATOM   2265  HA  VAL A 140       9.008  63.989  19.144   1.0   0.0           H
ATOM   2266  HB  VAL A 140       7.224  65.101  19.973   1.0   0.0           H
ATOM   2267 HG11 VAL A 140       6.074  63.930  21.764   1.0   0.0           H
ATOM   2268 HG12 VAL A 140       7.130  62.960  21.946   1.0   0.0           H
ATOM   2269 HG13 VAL A 140       6.371  63.010  20.690   1.0   0.0           H
ATOM   2270 HG21 VAL A 140       7.238  66.016  22.518   1.0   0.0           H
ATOM   2271 HG22 VAL A 140       8.417  66.665  22.033   1.0   0.0           H
ATOM   2272 HG23 VAL A 140       8.523  65.490  22.863   1.0   0.0           H
ATOM   2273  N   TYR A 141       9.412  66.087  18.370   1.0   0.0           N
ATOM   2274  CA  TYR A 141      10.096  67.225  17.773   1.0   0.0           C
ATOM   2275  CB  TYR A 141       9.809  67.288  16.271   1.0   0.0           C
ATOM   2276  CG  TYR A 141      10.513  68.421  15.546   1.0   0.0           C
ATOM   2277  CD1 TYR A 141      11.908  68.471  15.475   1.0   0.0           C
ATOM   2278  CE1 TYR A 141      12.563  69.501  14.811   1.0   0.0           C
ATOM   2279  CZ  TYR A 141      11.820  70.493  14.209   1.0   0.0           C
ATOM   2280  OH  TYR A 141      12.458  71.509  13.544   1.0   0.0           O
ATOM   2281  CE2 TYR A 141      10.438  70.464  14.261   1.0   0.0           C
ATOM   2282  CD2 TYR A 141       9.792  69.434  14.922   1.0   0.0           C
ATOM   2283  C   TYR A 141       9.660  68.515  18.454   1.0   0.0           C
ATOM   2284  O   TYR A 141       8.463  68.722  18.677   1.0   0.0           O
ATOM   2285  H   TYR A 141       9.698  65.240  17.923   1.0   0.0           H
ATOM   2286  HA  TYR A 141      10.981  67.231  18.239   1.0   0.0           H
ATOM   2287  HB2 TYR A 141       9.912  66.293  16.260   1.0   0.0           H
ATOM   2288  HB3 TYR A 141       9.023  66.700  16.462   1.0   0.0           H
ATOM   2289  HD1 TYR A 141      12.446  67.750  15.911   1.0   0.0           H
ATOM   2290  HE1 TYR A 141      13.562  69.522  14.771   1.0   0.0           H
ATOM   2291  HH  TYR A 141      13.457  71.530  13.506   1.0   0.0           H
ATOM   2292  HE2 TYR A 141       9.906  71.187  13.821   1.0   0.0           H
ATOM   2293  HD2 TYR A 141       8.793  69.418  14.952   1.0   0.0           H
ATOM   2294  N   ASP A 142      10.642  69.351  18.794   1.0   0.0           N
ATOM   2295  CA  ASP A 142      10.388  70.679  19.299   1.0   0.0           C
ATOM   2296  CB  ASP A 142      11.320  71.011  20.455   1.0   0.0           C
ATOM   2297  CG  ASP A 142      11.029  72.380  21.063   1.0   0.0           C
ATOM   2298  OD1 ASP A 142       9.850  72.834  21.022   1.0   0.0           O
ATOM   2299  OD2 ASP A 142      11.981  73.001  21.602   1.0   0.0           O
ATOM   2300  C   ASP A 142      10.638  71.651  18.166   1.0   0.0           C
ATOM   2301  O   ASP A 142      11.786  71.877  17.807   1.0   0.0           O
ATOM   2302  H   ASP A 142      10.029  69.133  18.034   1.0   0.0           H
ATOM   2303  HA  ASP A 142       9.395  70.800  19.288   1.0   0.0           H
ATOM   2304  HB2 ASP A 142      11.190  70.088  20.818   1.0   0.0           H
ATOM   2305  HB3 ASP A 142      11.954  70.306  20.137   1.0   0.0           H
ATOM   2306  N   PRO A 143       9.569  72.236  17.595   1.0   0.0           N
ATOM   2307  CA  PRO A 143       9.710  73.196  16.494   1.0   0.0           C
ATOM   2308  CB  PRO A 143       8.270  73.643  16.242   1.0   0.0           C
ATOM   2309  CG  PRO A 143       7.439  72.481  16.652   1.0   0.0           C
ATOM   2310  CD  PRO A 143       8.158  71.874  17.823   1.0   0.0           C
ATOM   2311  C   PRO A 143      10.600  74.415  16.780   1.0   0.0           C
ATOM   2312  O   PRO A 143      11.063  75.069  15.840   1.0   0.0           O
ATOM   2313  HA  PRO A 143      10.359  72.814  15.836   1.0   0.0           H
ATOM   2314  HB2 PRO A 143       8.227  74.471  16.800   1.0   0.0           H
ATOM   2315  HB3 PRO A 143       8.290  74.304  15.492   1.0   0.0           H
ATOM   2316  HG2 PRO A 143       6.562  72.939  16.797   1.0   0.0           H
ATOM   2317  HG3 PRO A 143       6.957  72.194  15.824   1.0   0.0           H
ATOM   2318  HD2 PRO A 143       7.708  72.314  18.600   1.0   0.0           H
ATOM   2319  HD3 PRO A 143       7.681  71.046  18.120   1.0   0.0           H
ATOM   2320  N   ILE A 144      10.845  74.710  18.051   1.0   0.0           N
ATOM   2321  CA  ILE A 144      11.613  75.881  18.428   1.0   0.0           C
ATOM   2322  CB  ILE A 144      11.110  76.435  19.773   1.0   0.0           C
ATOM   2323  CG1 ILE A 144       9.596  76.691  19.688   1.0   0.0           C
ATOM   2324  CD1 ILE A 144       8.841  76.314  20.949   1.0   0.0           C
ATOM   2325  CG2 ILE A 144      11.859  77.714  20.151   1.0   0.0           C
ATOM   2326  C   ILE A 144      13.110  75.568  18.494   1.0   0.0           C
ATOM   2327  O   ILE A 144      13.902  76.141  17.734   1.0   0.0           O
ATOM   2328  H   ILE A 144       9.867  74.914  18.008   1.0   0.0           H
ATOM   2329  HA  ILE A 144      11.719  76.350  17.551   1.0   0.0           H
ATOM   2330  HB  ILE A 144      11.574  75.730  20.309   1.0   0.0           H
ATOM   2331 HG12 ILE A 144       9.803  77.570  19.258   1.0   0.0           H
ATOM   2332 HG13 ILE A 144       9.704  76.730  18.695   1.0   0.0           H
ATOM   2333 HD11 ILE A 144       7.857  76.480  20.894   1.0   0.0           H
ATOM   2334 HD12 ILE A 144       8.821  76.704  21.870   1.0   0.0           H
ATOM   2335 HD13 ILE A 144       8.634  75.435  21.379   1.0   0.0           H
ATOM   2336 HG21 ILE A 144      11.532  78.074  21.025   1.0   0.0           H
ATOM   2337 HG22 ILE A 144      11.816  78.619  19.727   1.0   0.0           H
ATOM   2338 HG23 ILE A 144      12.814  77.828  20.426   1.0   0.0           H
ATOM   2339  N   SER A 145      13.496  74.671  19.400   1.0   0.0           N
ATOM   2340  CA  SER A 145      14.893  74.230  19.515   1.0   0.0           C
ATOM   2341  CB  SER A 145      15.105  73.410  20.792   1.0   0.0           C
ATOM   2342  OG  SER A 145      14.600  72.088  20.658   1.0   0.0           O
ATOM   2343  C   SER A 145      15.278  73.389  18.313   1.0   0.0           C
ATOM   2344  O   SER A 145      16.454  73.236  17.998   1.0   0.0           O
ATOM   2345  H   SER A 145      13.242  75.226  20.193   1.0   0.0           H
ATOM   2346  HA  SER A 145      15.426  75.023  19.221   1.0   0.0           H
ATOM   2347  HB2 SER A 145      16.063  73.686  20.876   1.0   0.0           H
ATOM   2348  HB3 SER A 145      15.163  74.212  21.387   1.0   0.0           H
ATOM   2349  HG  SER A 145      14.738  71.553  21.491   1.0   0.0           H
ATOM   2350  N   LYS A 146      14.258  72.824  17.668   1.0   0.0           N
ATOM   2351  CA  LYS A 146      14.411  72.004  16.467   1.0   0.0           C
ATOM   2352  CB  LYS A 146      15.031  72.815  15.321   1.0   0.0           C
ATOM   2353  CG  LYS A 146      14.410  74.206  15.176   1.0   0.0           C
ATOM   2354  CD  LYS A 146      14.198  74.621  13.721   1.0   0.0           C
ATOM   2355  CE  LYS A 146      15.506  74.934  13.007   1.0   0.0           C
ATOM   2356  NZ  LYS A 146      16.266  76.024  13.684   1.0   0.0           N
ATOM   2357  C   LYS A 146      15.190  70.729  16.777   1.0   0.0           C
ATOM   2358  O   LYS A 146      15.858  70.175  15.910   1.0   0.0           O
ATOM   2359  H   LYS A 146      13.748  73.660  17.465   1.0   0.0           H
ATOM   2360  HA  LYS A 146      13.561  71.481  16.531   1.0   0.0           H
ATOM   2361  HB2 LYS A 146      15.937  72.518  15.622   1.0   0.0           H
ATOM   2362  HB3 LYS A 146      15.393  71.964  14.941   1.0   0.0           H
ATOM   2363  HG2 LYS A 146      13.771  73.970  15.908   1.0   0.0           H
ATOM   2364  HG3 LYS A 146      14.679  74.390  16.121   1.0   0.0           H
ATOM   2365  HD2 LYS A 146      13.549  73.869  13.608   1.0   0.0           H
ATOM   2366  HD3 LYS A 146      13.239  74.816  13.925   1.0   0.0           H
ATOM   2367  HE2 LYS A 146      15.719  73.960  12.931   1.0   0.0           H
ATOM   2368  HE3 LYS A 146      15.177  74.508  12.164   1.0   0.0           H
ATOM   2369  HZ1 LYS A 146      17.125  76.230  13.215   1.0   0.0           H
ATOM   2370  HZ2 LYS A 146      16.729  76.024  14.571   1.0   0.0           H
ATOM   2371  HZ3 LYS A 146      16.053  76.998  13.760   1.0   0.0           H
ATOM   2372  N   GLY A 147      15.074  70.268  18.024   1.0   0.0           N
ATOM   2373  CA  GLY A 147      15.648  69.005  18.464   1.0   0.0           C
ATOM   2374  C   GLY A 147      14.537  68.022  18.785   1.0   0.0           C
ATOM   2375  O   GLY A 147      13.355  68.368  18.773   1.0   0.0           O
ATOM   2376  H   GLY A 147      15.806  70.916  17.813   1.0   0.0           H
ATOM   2377  HA2 GLY A 147      16.292  68.957  17.700   1.0   0.0           H
ATOM   2378  HA3 GLY A 147      16.534  69.392  18.719   1.0   0.0           H
ATOM   2379  N   GLN A 148      14.928  66.794  19.097   1.0   0.0           N
ATOM   2380  CA  GLN A 148      13.994  65.768  19.544   1.0   0.0           C
ATOM   2381  CB  GLN A 148      14.238  64.478  18.771   1.0   0.0           C
ATOM   2382  CG  GLN A 148      13.811  64.561  17.305   1.0   0.0           C
ATOM   2383  CD  GLN A 148      13.690  63.213  16.584   1.0   0.0           C
ATOM   2384  OE1 GLN A 148      14.111  63.083  15.431   1.0   0.0           O
ATOM   2385  NE2 GLN A 148      13.132  62.207  17.255   1.0   0.0           N
ATOM   2386  C   GLN A 148      14.129  65.545  21.063   1.0   0.0           C
ATOM   2387  O   GLN A 148      15.203  65.171  21.555   1.0   0.0           O
ATOM   2388  H   GLN A 148      14.840  66.939  18.111   1.0   0.0           H
ATOM   2389  HA  GLN A 148      13.146  66.283  19.671   1.0   0.0           H
ATOM   2390  HB2 GLN A 148      15.134  64.417  19.211   1.0   0.0           H
ATOM   2391  HB3 GLN A 148      14.311  64.008  19.651   1.0   0.0           H
ATOM   2392  HG2 GLN A 148      13.165  65.260  17.612   1.0   0.0           H
ATOM   2393  HG3 GLN A 148      14.064  65.525  17.387   1.0   0.0           H
ATOM   2394 HE21 GLN A 148      12.791  62.312  18.189   1.0   0.0           H
ATOM   2395 HE22 GLN A 148      13.051  61.315  16.809   1.0   0.0           H
ATOM   2396  N   LEU A 149      13.042  65.800  21.793   1.0   0.0           N
ATOM   2397  CA  LEU A 149      13.048  65.813  23.255   1.0   0.0           C
ATOM   2398  CB  LEU A 149      12.289  67.033  23.768   1.0   0.0           C
ATOM   2399  CG  LEU A 149      12.700  68.400  23.223   1.0   0.0           C
ATOM   2400  CD1 LEU A 149      11.779  69.484  23.770   1.0   0.0           C
ATOM   2401  CD2 LEU A 149      14.150  68.703  23.561   1.0   0.0           C
ATOM   2402  C   LEU A 149      12.357  64.586  23.787   1.0   0.0           C
ATOM   2403  O   LEU A 149      11.346  64.173  23.238   1.0   0.0           O
ATOM   2404  H   LEU A 149      13.501  66.615  21.440   1.0   0.0           H
ATOM   2405  HA  LEU A 149      13.967  65.558  23.554   1.0   0.0           H
ATOM   2406  HB2 LEU A 149      11.509  66.412  23.699   1.0   0.0           H
ATOM   2407  HB3 LEU A 149      12.007  66.364  24.455   1.0   0.0           H
ATOM   2408  HG  LEU A 149      12.293  68.207  22.330   1.0   0.0           H
ATOM   2409 HD11 LEU A 149      12.048  70.379  23.413   1.0   0.0           H
ATOM   2410 HD12 LEU A 149      11.717  69.822  24.709   1.0   0.0           H
ATOM   2411 HD13 LEU A 149      10.813  69.634  23.558   1.0   0.0           H
ATOM   2412 HD21 LEU A 149      14.754  67.992  23.202   1.0   0.0           H
ATOM   2413 HD22 LEU A 149      14.599  68.687  24.454   1.0   0.0           H
ATOM   2414 HD23 LEU A 149      14.735  69.432  23.206   1.0   0.0           H
ATOM   2415  N   TRP A 150      12.874  64.025  24.878   1.0   0.0           N
ATOM   2416  CA  TRP A 150      12.250  62.858  25.511   1.0   0.0           C
ATOM   2417  CB  TRP A 150      13.182  62.232  26.538   1.0   0.0           C
ATOM   2418  CG  TRP A 150      14.404  61.614  26.035   1.0   0.0           C
ATOM   2419  CD1 TRP A 150      15.662  62.143  26.067   1.0   0.0           C
ATOM   2420  NE1 TRP A 150      16.559  61.252  25.536   1.0   0.0           N
ATOM   2421  CE2 TRP A 150      15.886  60.120  25.156   1.0   0.0           C
ATOM   2422  CD2 TRP A 150      14.527  60.310  25.469   1.0   0.0           C
ATOM   2423  CE3 TRP A 150      13.617  59.284  25.182   1.0   0.0           C
ATOM   2424  CZ3 TRP A 150      14.093  58.112  24.599   1.0   0.0           C
ATOM   2425  CH2 TRP A 150      15.455  57.952  24.303   1.0   0.0           C
ATOM   2426  CZ2 TRP A 150      16.363  58.942  24.570   1.0   0.0           C
ATOM   2427  C   TRP A 150      10.989  63.254  26.268   1.0   0.0           C
ATOM   2428  O   TRP A 150      11.018  64.211  27.042   1.0   0.0           O
ATOM   2429  H   TRP A 150      13.702  63.765  24.381   1.0   0.0           H
ATOM   2430  HA  TRP A 150      11.822  62.310  24.792   1.0   0.0           H
ATOM   2431  HB2 TRP A 150      12.968  63.040  27.087   1.0   0.0           H
ATOM   2432  HB3 TRP A 150      12.370  62.265  27.121   1.0   0.0           H
ATOM   2433  HD1 TRP A 150      15.895  63.047  26.425   1.0   0.0           H
ATOM   2434  HE1 TRP A 150      17.543  61.403  25.441   1.0   0.0           H
ATOM   2435  HE3 TRP A 150      12.645  59.391  25.392   1.0   0.0           H
ATOM   2436  HZ3 TRP A 150      13.454  57.373  24.388   1.0   0.0           H
ATOM   2437  HH2 TRP A 150      15.769  57.098  23.889   1.0   0.0           H
ATOM   2438  HZ2 TRP A 150      17.333  58.827  24.354   1.0   0.0           H
ATOM   2439  N   VAL A 151       9.893  62.526  26.080   1.0   0.0           N
ATOM   2440  CA  VAL A 151       8.727  62.740  26.920   1.0   0.0           C
ATOM   2441  CB  VAL A 151       7.468  62.054  26.379   1.0   0.0           C
ATOM   2442  CG1 VAL A 151       6.331  62.154  27.393   1.0   0.0           C
ATOM   2443  CG2 VAL A 151       7.065  62.653  25.041   1.0   0.0           C
ATOM   2444  C   VAL A 151       9.047  62.165  28.281   1.0   0.0           C
ATOM   2445  O   VAL A 151       9.517  61.028  28.386   1.0   0.0           O
ATOM   2446  H   VAL A 151       9.681  62.906  25.180   1.0   0.0           H
ATOM   2447  HA  VAL A 151       8.794  63.693  27.216   1.0   0.0           H
ATOM   2448  HB  VAL A 151       7.833  61.142  26.567   1.0   0.0           H
ATOM   2449 HG11 VAL A 151       5.509  61.706  27.040   1.0   0.0           H
ATOM   2450 HG12 VAL A 151       5.822  62.968  27.673   1.0   0.0           H
ATOM   2451 HG13 VAL A 151       6.252  61.700  28.281   1.0   0.0           H
ATOM   2452 HG21 VAL A 151       6.243  62.205  24.688   1.0   0.0           H
ATOM   2453 HG22 VAL A 151       7.534  62.579  24.161   1.0   0.0           H
ATOM   2454 HG23 VAL A 151       6.700  63.565  24.853   1.0   0.0           H
ATOM   2455  N   VAL A 152       8.779  62.956  29.319   1.0   0.0           N
ATOM   2456  CA  VAL A 152       9.034  62.579  30.714   1.0   0.0           C
ATOM   2457  CB  VAL A 152      10.279  63.285  31.273   1.0   0.0           C
ATOM   2458  CG1 VAL A 152      10.574  62.794  32.677   1.0   0.0           C
ATOM   2459  CG2 VAL A 152      11.465  63.044  30.363   1.0   0.0           C
ATOM   2460  C   VAL A 152       7.845  62.960  31.584   1.0   0.0           C
ATOM   2461  O   VAL A 152       7.296  64.044  31.451   1.0   0.0           O
ATOM   2462  H   VAL A 152       9.560  62.706  28.747   1.0   0.0           H
ATOM   2463  HA  VAL A 152       8.870  61.593  30.752   1.0   0.0           H
ATOM   2464  HB  VAL A 152       9.794  64.147  31.423   1.0   0.0           H
ATOM   2465 HG11 VAL A 152      11.384  63.253  33.041   1.0   0.0           H
ATOM   2466 HG12 VAL A 152      10.898  61.889  32.953   1.0   0.0           H
ATOM   2467 HG13 VAL A 152      10.042  62.960  33.507   1.0   0.0           H
ATOM   2468 HG21 VAL A 152      12.275  63.503  30.727   1.0   0.0           H
ATOM   2469 HG22 VAL A 152      11.605  63.402  29.440   1.0   0.0           H
ATOM   2470 HG23 VAL A 152      11.950  62.182  30.213   1.0   0.0           H
ATOM   2471  N   VAL A 153       7.467  62.060  32.476   1.0   0.0           N
ATOM   2472  CA  VAL A 153       6.343  62.269  33.357   1.0   0.0           C
ATOM   2473  CB  VAL A 153       5.461  61.026  33.391   1.0   0.0           C
ATOM   2474  CG1 VAL A 153       4.169  61.317  34.123   1.0   0.0           C
ATOM   2475  CG2 VAL A 153       5.168  60.583  31.977   1.0   0.0           C
ATOM   2476  C   VAL A 153       6.853  62.532  34.756   1.0   0.0           C
ATOM   2477  O   VAL A 153       7.923  62.053  35.135   1.0   0.0           O
ATOM   2478  H   VAL A 153       7.130  61.886  31.551   1.0   0.0           H
ATOM   2479  HA  VAL A 153       6.019  63.202  33.200   1.0   0.0           H
ATOM   2480  HB  VAL A 153       6.033  60.543  34.054   1.0   0.0           H
ATOM   2481 HG11 VAL A 153       3.590  60.502  34.145   1.0   0.0           H
ATOM   2482 HG12 VAL A 153       3.416  61.905  33.828   1.0   0.0           H
ATOM   2483 HG13 VAL A 153       4.044  61.495  35.099   1.0   0.0           H
ATOM   2484 HG21 VAL A 153       4.589  59.768  31.999   1.0   0.0           H
ATOM   2485 HG22 VAL A 153       5.826  60.193  31.333   1.0   0.0           H
ATOM   2486 HG23 VAL A 153       4.596  61.066  31.314   1.0   0.0           H
ATOM   2487  N   GLY A 154       6.087  63.271  35.544   1.0   0.0           N
ATOM   2488  CA  GLY A 154       6.466  63.480  36.925   1.0   0.0           C
ATOM   2489  C   GLY A 154       5.317  63.783  37.844   1.0   0.0           C
ATOM   2490  O   GLY A 154       4.210  64.125  37.410   1.0   0.0           O
ATOM   2491  H   GLY A 154       6.852  63.069  34.932   1.0   0.0           H
ATOM   2492  HA2 GLY A 154       7.077  62.694  36.830   1.0   0.0           H
ATOM   2493  HA3 GLY A 154       7.397  63.623  36.588   1.0   0.0           H
ATOM   2494  N   THR A 155       5.588  63.634  39.133   1.0   0.0           N
ATOM   2495  CA  THR A 155       4.707  64.153  40.176   1.0   0.0           C
ATOM   2496  CB  THR A 155       3.503  63.216  40.452   1.0   0.0           C
ATOM   2497  OG1 THR A 155       2.549  63.895  41.273   1.0   0.0           O
ATOM   2498  CG2 THR A 155       3.939  61.931  41.116   1.0   0.0           C
ATOM   2499  C   THR A 155       5.517  64.435  41.446   1.0   0.0           C
ATOM   2500  O   THR A 155       6.701  64.076  41.553   1.0   0.0           O
ATOM   2501  H   THR A 155       5.059  63.450  38.304   1.0   0.0           H
ATOM   2502  HA  THR A 155       4.745  65.124  39.939   1.0   0.0           H
ATOM   2503  HB  THR A 155       2.948  63.315  39.626   1.0   0.0           H
ATOM   2504  HG1 THR A 155       1.772  63.291  41.451   1.0   0.0           H
ATOM   2505 HG21 THR A 155       3.162  61.327  41.294   1.0   0.0           H
ATOM   2506 HG22 THR A 155       4.296  61.779  42.038   1.0   0.0           H
ATOM   2507 HG23 THR A 155       4.473  61.171  40.745   1.0   0.0           H
ATOM   2508  N   SER A 156       4.864  65.101  42.389   1.0   0.0           N
ATOM   2509  CA  SER A 156       5.468  65.449  43.662   1.0   0.0           C
ATOM   2510  CB  SER A 156       5.903  66.913  43.657   1.0   0.0           C
ATOM   2511  OG  SER A 156       4.772  67.788  43.577   1.0   0.0           O
ATOM   2512  C   SER A 156       4.446  65.242  44.756   1.0   0.0           C
ATOM   2513  O   SER A 156       3.238  65.222  44.501   1.0   0.0           O
ATOM   2514  H   SER A 156       5.540  65.238  41.665   1.0   0.0           H
ATOM   2515  HA  SER A 156       6.030  64.667  43.932   1.0   0.0           H
ATOM   2516  HB2 SER A 156       6.517  66.800  44.438   1.0   0.0           H
ATOM   2517  HB3 SER A 156       6.825  66.729  43.315   1.0   0.0           H
ATOM   2518  HG  SER A 156       5.057  68.747  43.574   1.0   0.0           H
ATOM   2519  N   ALA A 157       4.939  65.106  45.981   1.0   0.0           N
ATOM   2520  CA  ALA A 157       4.073  65.023  47.163   1.0   0.0           C
ATOM   2521  CB  ALA A 157       3.720  63.573  47.466   1.0   0.0           C
ATOM   2522  C   ALA A 157       4.711  65.662  48.378   1.0   0.0           C
ATOM   2523  O   ALA A 157       5.930  65.590  48.559   1.0   0.0           O
ATOM   2524  H   ALA A 157       5.171  66.058  45.782   1.0   0.0           H
ATOM   2525  HA  ALA A 157       3.441  65.785  47.021   1.0   0.0           H
ATOM   2526  HB1 ALA A 157       3.299  63.151  46.663   1.0   0.0           H
ATOM   2527  HB2 ALA A 157       3.012  63.238  48.087   1.0   0.0           H
ATOM   2528  HB3 ALA A 157       4.352  62.811  47.608   1.0   0.0           H
ATOM   2529  OXT ALA A 157       4.121  65.605  49.183   1.0   0.0           O
TER
ENDMDL
END

```


模拟的输出结果为：



```
$ python3 test_openmm.py
#"Step","Potential Energy (kJ/mole)","Temperature (K)"
1000,-9033.825500488281,301.92065801292995
2000,-9268.240600585938,302.10168662116087
3000,-9273.523742675781,309.1021412145269
4000,-9472.859832763672,304.66424401527763
5000,-9340.563781738281,302.06315167179406
6000,-9413.580932617188,307.51769513484544
7000,-9418.724060058594,302.3040982815063
8000,-9683.616943359375,298.0298541650781
9000,-9539.312561035156,296.56061774392634
10000,-9480.7880859375,304.80073801952597

```

同时会在当前路径下保存一个`output.pdb`的最终构象。优化前后的构想对比如下图所示：



![](https://img2024.cnblogs.com/blog/2277440/202411/2277440-20241122150610657-277868320.png)

这样就完成了一个简单的能量极小化过程。


# 报错排查


这里提供一些有可能遇到的报错问题排查：



```
ValueError: Requested periodic boundary conditions for a Topology that does not specify periodic box dimensions

```

这个问题是由于力场中设置了PME而实际上没有Box输入导致的，如果需要模拟纯蛋白体系，就要在createSystem的时候把PME的选项关掉。



```
ValueError: coordinate "xxx" could not be represented in a width-8 field

```

这是由于在模拟的过程中，氢键被拉的太长而断裂所导致的，可以在createSystem的时候加上氢键的constraints。



```
ValueError: No template found for residue xxx.

```

这种问题经常出现在`CYS`这个residue上，实际使用的话，可以把已有文件中的CYS删掉，然后用pdbfixer重构。更多的常见问题，可以参考本文的参考链接3中的内容。


# 总结概要


本文介绍了AlphaFold2中所使用到的开源分子动力学模拟软件OpenMM的安装和基础使用方法，其中包含了pdbfixer蛋白质构象文件修复工具的介绍和一个真空蛋白体系的能量极小化示例，并且提供了一些有可能在OpenMM的安装和使用过程中遇到的问题和解决方法。


# 版权声明


本文首发链接为：[https://github.com/dechinphy/p/openmm.html](https://github.com)


作者ID：DechinPhy


更多原著文章：[https://github.com/dechinphy/](https://github.com)


请博主喝咖啡：[https://github.com/dechinphy/gallery/image/379634\.html](https://github.com):[FlowerCloud机场](https://hanlianfangzhi.com)


# 参考链接


1. [https://htmlpreview.github.io/?https://github.com/openmm/pdbfixer/blob/master/Manual.html](https://github.com)
2. [https://zhuanlan.zhihu.com/p/348312690](https://github.com)
3. [https://github.com/openmm/openmm/wiki/Frequently\-Asked\-Questions\#template](https://github.com)


