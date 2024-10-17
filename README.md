# DATASET
## Human3.6M
训练
```
python main.py -k cpn_ft_h36m_dbb -c checkpoint/model_h36m -gpu 0,1 --nolog
```
测试
```
python main.py -k cpn_ft_h36m_dbb -c checkpoint/model_h36m -gpu 0,1 --nolog --evaluate best_epoch_20_10.bin -num_proposals 20 -sampling_timesteps 10 -b 4
```
`best_epoch_20_10.bin`只会记录一次，表示从训练开始到当前训练周期内损失值最小，并且是在特定区间内的权重文件。具体的，特定区间设定为：

> if epoch >= args.save_emin and args.save_lmin <= losses_3d_valid[-1][0] * 1000 <= args.save_lmax and flag_best_20_10 == False:
    `epoch >= 70 and 41.10 <= losses_3d_valid[-1][0] * 1000 <= 41.25`

## MPI-INF-3DHP
训练
```
python main_3dhp.py -c checkpoint/model_3dhp -gpu 0,1 --nolog
```
测试
```
python main_3dhp.py -c checkpoint/model_3dhp -gpu 0,1 --nolog --evaluate best_epoch_20_10.bin -num_proposals 20 -sampling_timesteps 10 -b 4
```

## HumanEva-I
训练
```
python main_humaneva_gt.py -c checkpoint/model_humaneva_gt -gpu 0,1 --nolog -e 1000
```
测试
```
python main_humaneva_gt.py -gpu 0,1 --nolog --evaluate best_epoch_1_1.bin --p2 --by-subject -e 1000
```
`best_epoch_1_1.bin`记得改为best_epoch_20_10.bin
