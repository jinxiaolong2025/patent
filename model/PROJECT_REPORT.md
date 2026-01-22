# 项目流程与结果总览

本文档汇总 `/Users/jinxiaolong/Desktop/project/model` 项目的数据、流程、模型与产出文件（图表、表格），

## 数据与文件结构
- 原始数据：`data/raw/row_data1.xlsx`（原始样本），`data/raw/row_data_augment.xlsx`（SMOGN 增强后导出的合并样本）。
- 处理后数据（按场景）：
  - 不做目标变换：`data/processed/data_cleaned.xlsx`、`data_standardized.xlsx`、`data_augment_cleaned.xlsx`、`data_augment_standardized.xlsx`。
  - 目标 log1p 变换：`data/processed/data_cleaned_log.xlsx`、`data_standardized_log.xlsx`、`data_cleaned_log_augment.xlsx`、`data_standardized_log_augment.xlsx`。
- Notebook：`notebooks/feature_selection*.ipynb`（特征筛选）与 `notebooks/model_traning_*.ipynb`（建模评估，k 折/留一、原始/增强、是否 log）。
- 结果文件夹：`results/feature_selection/`、`results/xgb/`、`results/lin_pls_rf/`。

## 方法流程
1) **缺失值与异常值处理**
   - KNNImputer 填补缺失。
   - IQR 检测离群，对命中列做 1%/99% 缩尾。
   - 处理 inf，剩余 NaN 以中位数填补。
2) **分布变换与标准化**
   - Yeo-Johnson 变换 + 标准化，目标可选 log1p 变换。
3) **数据增强**
   - SMOGN 针对高脂尾部生成合成样本（k=5，pert=0.02，balance，rel_thres=0.8，rel_coef=1.5，手工控制点以 P75 为阈值）。
   - 输出 `data/raw/row_data_augment.xlsx` 并进入后续标准化。
4) **特征筛选**
   - Spearman 绝对相关 & PLS VIP 归一化求均值排序。
   - RFE（RidgeCV）与 MI 作为交叉验证参考。
   - 固定前 6 特征用于精简模型。
5) **建模与验证**
   - 模型族：XGBoost、Ridge/Lasso/ElasticNet、PLS、RandomForest。
   - 评估：RepeatedKFold(5×5) 与 LOOCV；目标 raw vs log1p；全特征 vs 前 6。
   - 调参：XGB 网格搜索（深度、采样率、学习率、迭代轮次、正则）以及线性/PLS/RF 的小型网格。

## 特征筛选结果（前 6，按 Average 得分）
- `results/feature_selection/feature_selection_results_raw.xlsx`：Turbidity，Total photosynthetic pigments，pH，DO，TOC，Dry cell weight。
- `results/feature_selection/feature_selection_results_log.xlsx`：Total photosynthetic pigments，Turbidity，DO，Chroma，Dry cell weight，pH。
- `results/feature_selection/feature_selection_results_augment.xlsx`：DO，pH，nitrate nitrogen，UV254，Dry cell weight，BOD。
- `results/feature_selection/feature_selection_results_augment_log.xlsx`：pH，DO，nitrate nitrogen，Dry cell weight，Total photosynthetic pigments，UV254。

## 输出文件索引
- 特征筛选图表与表格：`results/feature_selection/`
  - 比较图：`feature_selection_comparison_*.png`（raw/log/augment/augment_log）。
  - 评分表：`feature_selection_results_*.xlsx`（同四个场景）。
- XGBoost 结果：`results/xgb/`
  - 全特征/前 6，raw/augment/log 以及 k 折、留一对比：`xgb_full_repeated_*.png`、`xgb_subset_repeated_*.png`、`xgb_loocv_*_{mae,rmse,r2}.png`。
  - 网格搜索最佳模型：`xgb_gridsearch_best_*.png`（k 折 / 留一，raw/augment/log 组合）。
- 线性/PLS/RF 结果：`results/lin_pls_rf/`
  - Repeated 5×5：`lin_pls_rf_full_*_{mae,rmse,r2}.png`、`lin_pls_rf_sub_*_{mae,rmse,r2}.png`。
  - LOOCV：`lin_pls_rf_loo_*_{mae,rmse,r2}.png`。
  - 网格搜索最佳：`lin_pls_rf_grid_best_*.png`（k 折 / 留一，raw/augment/log 组合）。







