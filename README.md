### ➢best_epoch_%d_%.2f.bin？

`best_epoch_1_1.bin`保存从训练开始到当前训练周期内损失值最小的权重文件，只记录一次；

`best_epoch_20_10.bin`保存从训练开始到当前训练周期内损失值最小,并且是在特定区间内的权重文件，只记录一次。具体的，特定区间设定为：
> if epoch >= args.save_emin and args.save_lmin <= losses_3d_valid[-1][0] * 1000 <= args.save_lmax and flag_best_20_10 == False:
    `epoch >= 70 and 41.10 <= losses_3d_valid[-1][0] * 1000 <= 41.25`

### ➢xxx_test_log_H20_K10.txt？

使用best_epoch_20_10.bin进行测试生成的结果。

# DATASET
### Human3.6M
默认epoch>=80则每20轮，生成一个best_epoch_{epoch}_{loss}.bin

训练
```
python main.py -k cpn_ft_h36m_dbb -c checkpoint/model_h36m -gpu 0,1 --nolog
```
测试
```
python main.py -k cpn_ft_h36m_dbb -c checkpoint/model_h36m -gpu 0,1 --nolog --evaluate best_epoch_1_1.bin -num_proposals 20 -sampling_timesteps 10 -b 4
```

### MPI-INF-3DHP
默认epoch>=80则生成best_epoch_20_10.bin

训练
```
python main_3dhp.py -c checkpoint/model_3dhp -gpu 0,1 --nolog
```
测试
```
python main_3dhp.py -c checkpoint/model_3dhp -gpu 0,1 --nolog --evaluate best_epoch_1_1.bin -num_proposals 20 -sampling_timesteps 10 -b 4
```

### HumanEva-I
默认epoch>=980则每20轮，生成一个best_epoch_{epoch}_{loss}.bin
训练
```
python main_humaneva_gt.py -c checkpoint/model_humaneva_gt -gpu 0,1 --nolog -e 1000 -lrd 0.998
```
测试
```
python main_humaneva_gt.py -gpu 0,1 --nolog --evaluate best_epoch_1_1.bin --p2 --by-subject
```
# 评价指标
| 指标       | 描述                                           | 作用                               |
|------------|-----------------------------------------------|------------------------------------|
| **MPJPE**  | 原始预测与真实坐标的误差，包含平移、旋转、缩放差异 | 评估姿态的整体预测误差             |
| **P-MPJPE**| 对预测的 3D 关节坐标进行 Procrustes 对齐后再计算误差，消除了平移、旋转、缩放的影响 | 更关注姿态形状的一致性，排除全局因素 |

# 命令说明（arguments.py）
| 命令                           | 默认值                        | 作用                                                       | 其他说明                                                    | 是否针对某个数据集  |
|--------------------------------|-------------------------------|------------------------------------------------------------|-------------------------------------------------------------|--------------------|
| `-d`, `--dataset`              | 'humaneva'                    | 指定目标数据集                                              | 例如：`h36m` 或 `humaneva`                                   | 是，h36m或humaneva |
| `-k`, `--keypoints`            | 'gt'                          | 指定使用的2D检测数据                                        | 使用 ground truth 或其他检测数据                             | 否                 |
| `-str`, `--subjects-train`     | 'Train/S1,Train/S2,Train/S3'   | 指定训练集的主体                                            | 多个主体之间用逗号分隔                                       | 否                 |
| `-ste`, `--subjects-test`      | 'Validate/S1,Validate/S2,S3'   | 指定测试集的主体                                            | 多个主体之间用逗号分隔                                       | 否                 |
| `--save_lmin`                  | 41.10                         | 保存最小损失值                                              |                                                             | 否                 |
| `--save_lmax`                  | 41.25                         | 保存最大损失值                                              |                                                             | 否                 |
| `--save_lmin_gt`               | 21.20                         | Ground truth 的最小损失值                                    |                                                             | 否                 |
| `--save_lmax_gt`               | 21.35                         | Ground truth 的最大损失值                                    |                                                             | 否                 |
| `--save_emin`                  | 70                            | 保存最小误差值                                              |                                                             | 否                 |
| `-sun`, `--subjects-unlabeled` | ''                            | 指定自监督中无标签的主体                                     | 用逗号分隔无标签主体                                         | 否                 |
| `-a`, `--actions`              | 'Walk,Jog,Box'                | 指定动作类型                                                | 用逗号分隔的动作列表，或者 * 表示所有动作                    | 否                 |
| `-c`, `--checkpoint`           | 'checkpoint/model_humaneva_gt' | 指定保存检查点的目录                                        |                                                             | 否                 |
| `-l`, `--log`                  | 'log/default'                 | 指定日志文件的目录                                          |                                                             | 否                 |
| `-cf`, `--checkpoint-frequency`| N=20                          | 每N个周期创建一次检查点(生成.bin文件）                                   |                                                             | 否                 |
| `-r`, `--resume`               | ''                            | 指定要恢复的检查点文件名                                    |                                                             | 否                 |
| `--nolog`                      | False                         | 禁用日志功能                                                |                                                             | 否                 |
| `--evaluate`                   | ''                            | 指定用于评估的检查点文件                                    |                                                             | 否                 |
| `--render`                     | False                         | 可视化特定的视频                                            |                                                             | 否                 |
| `--by-subject`                 | False                         | 按主体分类评估误差                                          |                                                             | 否                 |
| `--export-training-curves`      | False                         | 保存训练曲线为 .png 图片                                     |                                                             | 否                 |
| `-s`, `--stride`               | 243                           | 训练时使用的帧块大小                                         |                                                             | 否                 |
| `-e`, `--epochs`               | 100                           | 训练的周期数                                                |                                                             | 否                 |
| `-e_gt`, `--epochs_gt`         | 120                           | Ground truth 的训练周期数                                    |                                                             | 否                 |
| `-b`, `--batch-size`           | 1024                          | 批处理大小                                                  |                                                             | 否                 |
| `-drop`, `--dropout`           | 0.0                           | Dropout 概率                                                |                                                             | 否                 |
| `-lr`, `--learning-rate`       | 0.00006                       | 初始学习率                                                  |                                                             | 否                 |
| `-lrd`, `--lr-decay`           | 0.993                         | 每个周期的学习率衰减                                        |                                                             | 否                 |
| `--coverlr`                    | False                         | 恢复之前模型时覆盖学习率                                     |                                                             | 否                 |
| `-mloss`, `--min_loss`         | 100000                        | 恢复之前模型时分配的最小损失                                |                                                             | 否                 |
| `-no-da`, `--no-data-augmentation` | False                    | 禁用训练时的数据增强                                        |                                                             | 否                 |
| `-cs`                          | 512                           | 模型的通道大小，只用于Transformer模型                       |                                                             | 否                 |
| `-dep`                         | 8                             | 模型的深度                                                  |                                                             | 否                 |
| `-alpha`                       | 0.01                          | 用于wf_mpjpe的参数                                           |                                                             | 否                 |
| `-beta`                        | 2                             | 用于wf_mpjpe的参数                                           |                                                             | 否                 |
| `--postrf`                     | False                         | 使用后处理模块                                              |                                                             | 否                 |
| `--ftpostrf`                   | False                         | 微调后处理模块                                              |                                                             | 否                 |
| `-f`, `--number-of-frames`     | 243                           | 使用的帧数                                                   |                                                             | 否                 |
| `-gpu`                         | '3,4'                         | 分配要使用的GPU设备编号                                      |                                                             | 否                 |
| `--subset`                     | 1                             | 减少数据集大小的比例                                         |                                                             | 是，半监督数据集    |
| `--downsample`                 | 1                             | 按因子下采样帧率（用于半监督）                               |                                                             | 是，半监督数据集    |
| `--warmup`                     | 1                             | 半监督训练的预热周期数                                       |                                                             | 是，半监督数据集    |
| `--no-eval`                    | False                         | 禁用训练期间的评估                                           | 可以提升训练速度                                           | 否                 |
| `--dense`                      | False                         | 使用稠密卷积代替扩展卷积                                     |                                                             | 否                 |
| `--disable-optimizations`      | False                         | 禁用单帧预测的优化模型                                       |                                                             | 否                 |
| `--linear-projection`          | False                         | 半监督投影时仅使用线性系数                                   |                                                             | 是，半监督数据集    |
| `--no-bone-length`             | True                          | 禁用骨长项（半监督设置）                                     |                                                             | 是，半监督数据集    |
| `--no-proj`                    | False                         | 禁用半监督设置下的投影                                       |                                                             | 是，半监督数据集    |
| `--ft`                         | False                         | 使用ft 2D (仅用于检测关键点)                                 |                                                             | 是，检测关键点数据  |
| `--ftpath`                     | 'checkpoint/exp13_ft2d'        | 指定ft2d模型的检查点路径                                     |                                                             | 是，检测关键点数据  |
| `--ftchk`                      | 'epoch_330.pth'               | 指定ft2d模型检查点文件名                                     |                                                             | 是，检测关键点数据  |
| `--cond_pose_mask_prob`        | 0.1                           | 掩盖整个姿态的概率                                           |                                                             | 否                 |
| `--cond_part_mask_prob`        | 0.1                           | 掩盖部分姿态的概率                                           |                                                             | 否                 |
| `--cond_joint_mask_prob`       | 0.1                           | 掩盖关节的概率                                               |                                                             | 否                 |
| `--viz-subject`                | None                          | 渲染时指定的主体                                             | 可选                                                     | 否                 |
| `--viz-action`                 | None                          | 渲染时指定的动作                                             | 可选                                                     | 否                 |
| `--viz-camera`                 | 0                             | 渲染时指定的相机                                             | 可选                                                     | 否                 |
| `--viz-video`                  | None                          | 渲染时指定的视频路径                                         | 可选                                                     | 否                 |
| `--viz-skip`                   | 0                             | 渲染时跳过的视频帧数                                         | 可选                                                     | 否                 |
| `--viz-output`                 | None                          | 输出文件名（支持.gif 或 .mp4）                                | 可选                                                     | 否                 |
| `--viz-export`                 | None                          | 输出坐标文件名                                               | 可选                                                     | 否                 |
| `--viz-bitrate`                | 3000                          | 渲染视频的比特率                                             | 可选                                                     | 否                 |
| `--viz-no-ground-truth`        | False                         | 不显示真实姿态                                               | 可选                                                     | 否                 |
| `--viz-limit`                  | -1                            | 仅渲染前N帧                                                  | 可选                                                     | 否                 |
| `--viz-downsample`             | 1                             | 通过因子N下采样FPS                                           | 可选                                                     | 否                 |
| `--viz-size`                   | 5                             | 输出图像大小                                                 | 可选                                                     | 否                 |
| `--compare`                    | False                         | 是否与其他方法进行比较                                       | 例如与Poseformer比较                                      | 否                 |
| `--debug`                      | False                         | 调试模式                                                     | 可选                                                     | 否                 |
| `--p2`                         | False                         | 使用协议#2（即P-MPJPE）                                       | 可选                                                     | 否                 |
