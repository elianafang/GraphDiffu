# DSGDiffu: Dynamic and Static Prompt-Driven Diffusion with Graph Attention for 3D Human Pose Estimation

<img width="638" alt="image" src="https://github.com/user-attachments/assets/7209abaf-996f-43a2-aafb-3ad4ada07a39" />


## Datasets & Downloads

### A. Human3.6M

We set up the Human3.6M dataset in the same way as [VideoPose3D](https://github.com/facebookresearch/VideoPose3D/blob/master/DATASETS.md).  You can download the processed data from [here](https://drive.google.com/file/d/1FMgAf_I04GlweHMfgUKzB0CMwglxuwPe/view?usp=sharing).  `data_2d_h36m_gt.npz` is the ground truth of 2D keypoints. `data_2d_h36m_cpn_ft_h36m_dbb.npz` is the 2D keypoints obatined by [CPN](https://github.com/GengDavid/pytorch-cpn).  `data_3d_h36m.npz` is the ground truth of 3D human joints. Put them in the `./data` directory.

### B. MPI-INF-3DHP

We set up the MPI-INF-3DHP dataset following [P-STMO](https://github.com/paTRICK-swk/P-STMO). However, our training/testing data is different from theirs. They train and evaluate on 3D poses scaled to the height of the universal skeleton used by Human3.6M (officially called "univ_annot3"), while we use the ground truth 3D poses (officially called "annot3"). The former does not guarantee that the reprojection (used by the proposed JPMA) of the rescaled 3D poses is consistent with the 2D inputs, while the latter does. You can download our processed data from [here](https://drive.google.com/file/d/1zOM_CvLr4Ngv6Cupz1H-tt1A6bQPd_yg/view?usp=share_link). Put them in the `./data` directory. 



## Train & Test

### A. Human3.6M

(1) **Train GraphDiffu (using the 2D keypoints obtained by CPN as inputs):**

```bash
python main.py -d h36m -k cpn_ft_h36m_dbb -str S1,S5,S6,S7,S8 -ste S9,S11 -c checkpoint/model_h36m_cpn -gpu 0 -lrd 0.998 --nolog -e 100
```

**The corresponding test code:**

```
python main_gt.py -d h36m -k gt -str S1,S5,S6,S7,S8 -ste S9,S11 -c checkpoint/model_h36m_gt_heads=1_lrd0.998_s243_f243 -gpu 0 --nolog --evaluate best_epoch_1_1.bin --p2 -sampling_timesteps 10 -num_proposals 20 -b 4 -s 243 -f 243
```

(2) **Train GraphDiffu (using the ground truth 2D poses as inputs):**

```bash
python main_gt.py -d h36m -k gt -str S1,S5,S6,S7,S8 -ste S9,S11 -c checkpoint/model_h36m_gt -gpu 0 --nolog -lrd 0.998 -e 100
```

**The corresponding test code:**

```
python main_gt.py -d h36m -k gt -str S1,S5,S6,S7,S8 -ste S9,S11 -c checkpoint/model_h36m_gt_heads=1_lrd0.998_s1_f1 -gpu 0 --nolog --evaluate best_epoch_1_1.bin --p2 -sampling_timesteps 10 -num_proposals 20 -b 4 -s 1 -f 1
```



### B. MPI-INF-3DHP

**Train GraphDiffu (using the ground truth 2D poses as inputs):**

```bash
python main_3dhp.py -d 3dhp -c checkpoint/mpi_1_lr7e-5_lrd0.995 -gpu 0,1 --nolog -lrd 0.995 -lr 0.00007 -e 120
```

**Test GraphDiffu:**

```
python main_3dhp.py -d 3dhp -c checkpoint/mpi_1_lr7e-5_lrd0.995 -gpu 0,1 --nolog --evaluate best_epoch_1_1.bin -num_proposals 20 -sampling_timesteps 10 -b 4
```

After that, the predicted 3D poses under P-Best, P-Agg, J-Best, J-Agg settings are saved as four files (`.mat`) in `./checkpoint`. To get the MPJPE, AUC, PCK metrics, you can evaluate the predictions by running a Matlab script `./3dhp_test/test_util/mpii_test_predictions_ori_py.m` (you can change 'aggregation_mode' in line 29 to get results under different settings). Then, the evaluation results are saved in `./3dhp_test/test_util/mpii_3dhp_evaluation_sequencewise_ori_{setting name}_t{iteration index}.csv`. You can manually average the three metrics in these files over six sequences to get the final results.





## Experimental Environment

Trained on NVIDIA RTX 4090.

* pytorch >= 0.4.0
* matplotlib=3.1.0
* einops
* timm
* tensorboard
* torch-geometric 2.4.0
* CLIP
  
  

## Acknowledgement

Our code refers to the following repositories. We thank the authors for releasing their codes.

* [MotionDiffuse](https://github.com/mingyuan-zhang/MotionDiffuse)
* [D3DP](https://github.com/paTRICK-swk/D3DP)
* [MixSTE](https://github.com/JinluZhang1126/MixSTE)
* [FinePOSE](https://github.com/PKU-ICST-MIPL/FinePOSE_CVPR2024)
* [KTPFormer](https://github.com/JihuaPeng/KTPFormer)
