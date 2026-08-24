# ML-Driven-Prediction-of-Ransomware-Economic-Impact

📌 Project Overview

ML-Driven-Prediction-of-Ransomware-Economic-Impact is an Explainable AI–Driven Ransomware Economic Impact Prediction System designed to process ransomware-related network traffic, engineer relevant features, generate economic-impact labels, train and optimize machine learning regression models, evaluate model performance, and provide prediction capabilities through a REST API and command-line interface.

The repository contains the complete data-processing, feature-engineering, machine-learning, evaluation, explainability, and deployment workflow, together with datasets, reports, figures, trained models, and supporting artifacts.

🎯 Key Capabilities

🔍 Ransomware-related network traffic extraction from CICIDS2017 and CICIDS2018.

🧹 Data cleaning, schema standardisation, and preprocessing.

🏢 Sector-aware economic impact modelling.

⚙️ Feature engineering for temporal, behavioural, network, and categorical characteristics.

🤖 Machine learning regression using Linear Regression, Random Forest, XGBoost, and LightGBM.

🔧 Hyperparameter optimization using Optuna.

📊 Model evaluation using RMSE, MAE, R², and MAPE.

🧠 Explainable AI using SHAP.

🔬 Sensitivity and sector-level error analysis.

🚀 Model deployment through FastAPI and Uvicorn.

💻 Command-line prediction through the inference pipeline.

📁 Reproducible project configuration and structured data/model artifacts.

📂 Repository Documentation

The following section preserves the original project documentation and file descriptions while presenting them in a GitHub-friendly Markdown structure.

ML-Driven-Prediction-of-Ransomware-Economic-Impact

📂 Project Structure & File Descriptions
This section documents the purpose of each major configuration file, source module, dataset, report, and model artifact used in the Explainable AI–Driven Ransomware Economic Impact Prediction System.



The config.yaml file centralises all project settings in one place to ensure consistency, reproducibility, and easy configuration management.

It defines:

📁 Project paths (raw data, processed data, splits, outputs)
🎲 Random seed for reproducible experiments
🏢 Sector parameters (weights, criticality, hourly cost) used for economic modelling
💰 Synthetic label generation settings (downtime, ransom distribution, payment probability)
📊 Indirect cost and noise parameters
📦 Train/validation/test split ratios
✅ In short:
It allows you to modify system behaviour (data paths, sector assumptions, economic modelling parameters) without changing the source code, improving maintainability and experimental control.

###############
✅ Purpose of src/api.py
This file implements the REST API layer for the ransomware economic impact prediction system using FastAPI. It transforms the trained ML model into a production-ready service, enabling real-time predictions through HTTP requests.

It creates a FastAPI web application, defines a validated request schema using Pydantic, lazy-loads the trained prediction model, and exposes a /predict endpoint that returns estimated financial impact in USD. A /health endpoint is also provided to check service availability, and the application runs via Uvicorn for deployment.

✅ In short, src/api.py serves as the deployment interface that allows external users, security systems, or applications to interact with the trained model as a live prediction service.

###############

src/assign_sector.py

This script enriches a dataset of ransomware sessions by assigning each session to a simulated industry sector using weighted random selection, with 20% intentionally directed to high‑criticality sectors (Energy and Finance) to mimic targeted attacks. It then maps each sector to predefined criticality levels and estimated hourly financial impact, producing a business‑contextualized dataset for risk analysis and impact modeling.

###############
This src/config.py file centralizes project configuration by reading settings from a config.yaml file using PyYAML, loading it once into a singleton CONFIG object for easy imports. It then derives key filesystem paths (like PROJECT_ROOT, RAW_DIR, PROCESSED_DIR, and SPLITS_DIR) from the YAML values and ensures those directories exist by creating them if they’re missing, so the rest of the pipeline can reliably read/write data.

###############
src/eda_analysis.py
This file performs exploratory data analysis (EDA) on the ransomware impact dataset by generating key visualizations and statistics to understand the distribution of economic impact, sector-level differences, and feature correlations with the target variable. It automatically saves plots (histograms, boxplots, correlation heatmap, and top feature scatter plots) to support data understanding and modeling decisions.

###############
src/evaluate_model.py
This file evaluates the trained machine learning model on the test set and performs comprehensive post-modeling analysis, including computing performance metrics (RMSE, MAE, R², MAPE), generating SHAP interpretability plots, conducting sector-level error analysis, and running sensitivity testing by retraining the model without sector-related features. It saves all evaluation results, visualizations, and reports for final model assessment and validation.

###############

src/extract_ransomware.py

The script src/extract_ransomware.py serves as the first stage of the data pipeline. Its main purpose is to read the raw network traffic datasets, namely CICIDS2017 and CICIDS2018 in CSV format, filter them down to only the flows that are relevant to ransomware activity, standardize their column names, engineer a couple of basic features, and save the cleaned result as a single Parquet file called ransomware_sessions_raw.parquet for downstream processing. In short, it transforms raw CSVs into a filtered and unified set of ransomware-related flows stored in Parquet format.

The workflow proceeds in several steps. First, the script locates all CSV files inside the raw data directory and raises an error if none are found. It then streams each file in chunks of 100,000 rows to keep memory usage low, since IDS datasets can be very large. For every chunk, a schema-detection function determines whether the data follows the CICIDS2017 layout (with columns like Total Fwd Packets) or the CICIDS2018 layout (with columns like Tot Fwd Pkts), because the two datasets use slightly different naming conventions. Once the schema is known, the Label column is lowercased and matched against a list of ransomware-related keywords using a regular expression, and only the matching rows are kept. The original CICIDS columns are then renamed through a schema-specific mapping dictionary into a unified project-friendly scheme such as session_id, timestamp, dst_port, and attack_variant, which allows the rest of the pipeline to treat both datasets identically. After validating that all required columns are present, the script engineers two new features: flow_bytes_per_sec, computed as the total forward and backward bytes divided by flow duration, and flow_packets_per_sec, computed as the total forward and backward packets divided by flow duration, with the duration clipped at a small value to avoid division by zero. Finally, all retained chunks are concatenated, randomly shuffled using a fixed random seed for reproducibility, and written to a Parquet file.

The most important conceptual point concerns the list of keywords used for filtering. The CICIDS2017 and CICIDS2018 datasets do not contain a literal "Ransomware" label, because ransomware is an outcome—file encryption combined with extortion—rather than a single network signature. In real-world incidents, ransomware is delivered through a kill chain composed of several earlier attack stages, and the keywords chosen in this script act as proxies for those stages, capturing the network behaviors that typically precede, enable, or accompany a ransomware deployment. Brute force represents the initial access stage and is the most common ransomware entry vector, since attackers frequently brute-force RDP, SSH, or SMB credentials to gain a foothold, as seen in campaigns such as Ryuk, Conti, and LockBit. Web attack corresponds to initial access and exploitation through web applications, where vulnerabilities such as XSS, file upload flaws, or weak CMS configurations are abused to drop an initial loader or web shell that later fetches the ransomware payload. SQL injection is used to compromise backend servers, steal credentials, or pivot into the internal network, and it is a frequent precursor to double-extortion ransomware in which data is exfiltrated before encryption. Infiltration (and its misspelled variant "infilteration") corresponds to the execution and lateral-movement stage and directly represents the malware establishing presence inside the network and spreading, which is exactly what a ransomware loader does after initial access. Finally, bot traffic corresponds to command-and-control and delivery, because botnets such as Emotet, TrickBot, and QakBot are the primary delivery mechanisms for modern ransomware—TrickBot famously delivered Ryuk, and Emotet delivered Conti—so bot C2 traffic is a strong indicator of ransomware activity.

Taken together, the script treats these five behaviors as pre-ransomware or ransomware-enabling signals. The underlying modeling assumption is that if a flow exhibits brute-force, web-exploitation, SQL injection, infiltration, or botnet behavior, it belongs to the same attack family that produces ransomware incidents, and its network features can therefore be used to learn the patterns associated with ransomware-related activity. This is a reasonable and commonly used proxy in academic ransomware-detection work when a labeled ransomware dataset is unavailable, but it should be explicitly stated as a limitation in any report or thesis: the model is learning ransomware-precursor traffic rather than the encryption traffic itself. In summary, extract_ransomware.py converts heterogeneous raw IDS CSVs into a single, schema-unified, ransomware-relevant Parquet dataset by filtering on attack labels that correspond to stages of the ransomware kill chain, renaming columns to a common schema, and adding two throughput-based features, with the chosen keywords representing the delivery and intrusion precursors—brute force, web and SQLi exploitation, infiltration, and botnet command-and-control—that together approximate the network footprint of a ransomware campaign.

###############
src/feature_engineering.py
The script src/feature_engineering.py transforms the raw and split datasets by generating new, model-ready features that capture temporal, behavioral, and categorical patterns relevant to ransomware impact prediction. It extracts time-based attributes (such as hour of day, day of week, and weekend indicators) from timestamps, computes network traffic ratios (like average packet size and forward-to-backward byte/packet ratios), and categorizes destination ports into common service types (e.g., SMB, RDP, SSH). Additionally, it applies one-hot encoding to the sector column, introduces a synthetic sector vulnerability index, and creates a non-linear target transformation (impact_squared). Finally, it applies these identical transformations consistently across the full dataset and the train/validation/test splits, saving the enriched data as Parquet files for downstream modeling.

###############
src/generate_labels.py
The script src/generate_labels.py synthesizes the target variable for the machine learning model by simulating the financial impact of a ransomware attack on each network session. It generates realistic distributions for downtime hours, ransom demands, and payment probabilities, then calculates the total economic impact by combining direct operational losses (based on sector-specific hourly costs), expected ransom payments, and indirect business costs. Finally, it adds controlled Gaussian noise to the total impact to simulate real-world variability, applies a logarithmic transformation, and saves the fully labeled dataset in both CSV and Parquet formats for downstream feature engineering and modeling.

###############
src/optimize_model.py
The script src/optimize_model.py automates the hyperparameter tuning process for the LightGBM regression model using the Optuna optimization framework. It defines a comprehensive search space for key model parameters (such as learning rate, tree depth, and regularization) and evaluates each trial by minimizing the Root Mean Squared Error (RMSE) on the validation set. After identifying the optimal hyperparameter configuration, the script retrains the final model on the combined training and validation datasets to maximize generalization, and then saves the optimized model and study results for deployment.

###############
src/predict.py

The predict.py script implements the deployment stage of the ransomware economic impact prediction framework. It provides an end-to-end inference pipeline that transforms raw network traffic data into a machine learning-ready format and generates an estimated financial impact of a ransomware attack.

The module performs the following main tasks:

Loads the optimized trained machine learning model and supporting preprocessing components (scaler and feature configuration).

Validates incoming network flow data to ensure all required features are available.

Reapplies the same feature engineering process used during model training, including timestamp processing, ratio feature generation, and sector encoding.

Aligns input features with the exact feature structure used during model development to ensure consistent predictions.

Applies feature scaling using the previously fitted scaler.

Uses the trained model to predict the expected ransomware economic impact in USD.

Provides a command-line interface (CLI) for users to enter new network traffic characteristics and obtain real-time predictions.

This module ensures consistency between the training and prediction phases by reproducing the complete preprocessing and machine learning pipeline, enabling reliable estimation of the potential financial consequences of ransomware incidents.

###############
src/prepare_ml_data.py – Machine Learning Data Preparation Pipeline

The prepare_ml_data.py script prepares the engineered ransomware dataset for machine learning model development. It converts processed network traffic data into a model-ready format by separating input features and target values, removing irrelevant or leakage-prone attributes, applying feature scaling, and saving the prepared datasets for training and evaluation.

The module performs the following main tasks:

Loads the engineered training, validation, and test datasets generated during the feature engineering phase.

Removes non-predictive attributes, identifiers, categorical strings, and target-related features to prevent data leakage.

Separates the input features (X) from the prediction target (economic_impact_usd).

Selects numerical features, handles missing values, and prepares data in a format compatible with machine learning algorithms.

Fits a StandardScaler only on the training dataset and applies the same scaling transformation to validation and test datasets.

Saves the fitted scaler for future prediction and deployment stages.

Stores the prepared machine learning datasets as compressed NumPy files and human-readable Parquet files.

Preserves the feature names to ensure consistent feature ordering during model training and inference.

This script creates the final ML-ready dataset used in the model training phase, ensuring consistent preprocessing, preventing target leakage, and maintaining reproducibility between training and prediction workflows.

###############
src/split_data.py – Dataset Splitting and Stratification

The split_data.py script divides the labelled ransomware dataset into training, validation, and testing subsets while maintaining a balanced distribution of attack sectors across all splits. It ensures that the machine learning model is trained, validated, and evaluated using representative samples from each sector.

The module performs the following main tasks:

Loads the labelled ransomware impact dataset generated during the data preparation stage.

Applies stratified sampling based on the sector attribute to preserve sector distribution across datasets.

Splits the dataset into training, validation, and testing sets according to predefined ratios.

Uses a two-stage stratified splitting strategy:

First, separates the training set from the combined validation and testing data.

Second, divides the remaining data into validation and testing sets.

Saves the generated datasets as separate CSV files for subsequent feature engineering and machine learning preparation stages.

Verifies and reports the sector distribution within each split to ensure consistent representation.

This script ensures reliable model evaluation by preventing biased data splits and maintaining similar sector characteristics across training, validation, and test datasets.

###############
src/train_lightgbm.py

LightGBM Model Training Function

The train_lightgbm() function trains a LightGBM regression model to predict ransomware economic impact based on engineered network traffic features. It implements an optimized gradient boosting approach with validation monitoring and early stopping to improve model performance and prevent overfitting.

The function performs the following main tasks:

Initializes a LightGBM regression model with predefined hyperparameters, including learning rate, tree depth, number of leaves, and sampling parameters.

Trains the model using the prepared training dataset.

Evaluates model performance on the validation dataset using Root Mean Square Error (RMSE).

Applies early stopping to automatically terminate training when validation performance stops improving.

Records the optimal number of boosting rounds based on validation results.

Returns the trained LightGBM model for evaluation, comparison, and deployment.

This training approach enables efficient model optimization while improving generalization performance for ransomware economic impact prediction.

###############
src/train_models.py

This script implements the machine learning training pipeline for ransomware economic impact prediction. It loads the prepared datasets, trains multiple regression models including Linear Regression, Random Forest, XGBoost, and LightGBM, evaluates their performance using RMSE, MAE, R², and MAPE metrics, and selects the best-performing model based on validation results. The trained models and the final optimized model are saved for further evaluation and deployment.

###############
src/verify_downloads.py

This script verifies the availability and validity of the downloaded network traffic datasets before processing. It checks that the raw CSV files exist, confirms they follow the expected CICFlowMeter format (CICIDS2017/2018), and ensures that the dataset contains malicious traffic samples in addition to benign traffic. Successful verification confirms that the data is ready for the extraction and preprocessing stages.

###############
data/reports/sensitivity_results.txt
This sensitivity analysis evaluates the importance of sector-related features in the model. Three features were removed: sector_criticality, sector_hourly_cost, and sector_vulnerability_index. The model was retrained without these features and compared against the original model that used the full feature set. The goal was to measure how much predictive performance depends on sector context.

The results show a substantial degradation in performance when sector features are removed. With the full feature set, the model achieved an RMSE of 2,756,455 and an MAE of 1,936,175. After removing the sector features, RMSE increased to 20,315,367 and MAE increased to 12,216,351. This indicates a significant rise in prediction error, meaning the model becomes far less accurate without sector information.

The R² score also dropped sharply from 0.99 to 0.40. While the full model explains 99% of the variance in the target variable, the reduced model explains only 40%. This large decrease demonstrates that sector-related variables contribute heavily to the model’s explanatory power.

Overall, the analysis confirms that sector context is a critical driver of predictive performance. The model relies strongly on industry-specific characteristics, and removing these features results in a substantial loss of accuracy and model fit. Therefore, sector-related features should be retained in the final production model.

###############
data/reports/error_analysis.csv
The error_analysis.csv dataset contains network traffic session details utilized for analyzing model misclassifications and error patterns in ransomware detection. It includes granular flow logs featuring unique session identifiers (session_id), destination ports (dst_port), protocol types (protocol), precise activity timestamps (timestamp), and the overall duration of the network flow (flow_duration). This data serves as a diagnostic resource for debugging and refining machine learning models by isolating traffic characteristics associated with prediction errors.

###############
data/reports/high_error_cases.csv
This file, high_error_cases.csv, contains network flow records that resulted in the highest prediction errors or misclassifications during machine learning model evaluation. It tracks key network attributes—including session identifiers, destination ports, protocols, timestamps, and flow durations—to help diagnose model weaknesses, identify edge cases, and guide future feature engineering or training data refinements for improved ransomware detection accuracy.

###############
data/reports/shap_values_subset.csv
This dataset (shap_values_subset.csv) contains computed SHAP (SHapley Additive exPlanations) values derived from a machine learning model trained to detect and analyze ransomware behavior based on network traffic characteristics. Each row represents a specific network flow instance, while the columns correspond to key network features such as the destination port (dst_port), protocol (protocol), communication duration (flow_duration), and packet counts (total_fwd_packets, total_bwd_packets). The floating-point values within the matrix quantify the marginal contribution (both positive and negative) of each respective feature toward the model's final classification decision, serving as a critical component for model interpretability, feature importance evaluation, and explainable AI (XAI) workflows in cybersecurity diagnostics.

###############
data/reports/test_metrics.csv
This file (test_metrics.csv) records the performance evaluation metrics for a machine learning model, likely trained for regression tasks given the specific metrics reported. It contains a single row of evaluation results across four key metrics: a Root Mean Squared Error (RMSE) of approximately 2,756,455.24 and a Mean Absolute Error (MAE) of about 1,936,175.10, which quantify the average magnitude of the model's prediction errors. Despite these large absolute error values, the model achieves an exceptionally high $R^2$ (Coefficient of Determination) score of approximately 0.9890, indicating that the model explains roughly 98.9% of the variance in the target variable. Furthermore, the Mean Absolute Percentage Error (MAPE) is quite low at approximately 4.03%, demonstrating that the model's predictions deviate from the actual values by an average of only about 4% in relative terms.

###############
The figures located in data/reports/figures/ summarise exploratory analysis, model validation, and explainability results for the ransomware economic impact prediction system.

data/reports/figures/correlation_heatmap.png:
The correlation analysis in data/reports/figures/correlation_heatmap.png
illustrates the strongest relationships between economic impact and predictor variables. It highlights the dominant influence of downtime and sector-aware economic features while also revealing interdependencies among network flow variables.

data/reports/figures/error_by_sector.png:
Prediction stability across industries is shown in
data/reports/figures/error_by_sector.png
which demonstrates consistent absolute percentage error distributions across sectors, confirming robust cross-sector generalisation.

data/reports/figures/error_distribution.png:
Overall model accuracy is visualised in data/reports/figures/error_distribution.png
where most predictions fall within a low error range, with only a small tail of higher deviations.

data/reports/figures/feature_vs_target_top5.png:
Feature–target relationships for the most correlated variables are presented in data/reports/figures/feature_vs_target_top5.png
showing strong nonlinear growth between downtime and impact, along with clear monotonic effects from sector vulnerability and criticality.

data/reports/figures/sector_impact_boxplot.png:
Sector-level economic differences are displayed in data/reports/figures/sector_impact_boxplot.png
where Finance and Energy exhibit higher median and variance in losses, validating the need for sector-aware modelling.

data/reports/figures/sensitivity_comparison.png:
The importance of sector features is quantified in data/reports/figures/sensitivity_comparison.png
which shows significant degradation in RMSE and MAE when sector variables are removed from the model.

data/reports/figures/shap_bar_plot.png:
Global feature importance derived from SHAP analysis is shown in data/reports/figures/shap_bar_plot.png
ranking downtime and sector economic attributes as dominant contributors.

data/reports/figures/shap_summary_plot.png:
A more detailed explainability view is provided in data/reports/figures/shap_summary_plot.png
illustrating how high and low feature values influence individual predictions.

data/reports/figures/target_distribution.png:
the target variable distribution is shown in data/reports/figures/target_distribution.png
demonstrating the heavy right-skew of raw economic losses and the stabilising effect of log transformation.

Together, these visualisations confirm that the system is accurate, sector-aware, economically coherent, and fully explainable for deployment in ransomware risk assessment scenarios.

###############
data/models/optuna_study_results.csv
This file contains the hyperparameter tuning logs generated by Optuna during the optimization phase of the ransomware machine learning model (ransomware_ml). It records the performance metric (value) alongside complete execution metadata for each trial—including exact start and completion timestamps (datetime_start, datetime_complete) and the computational duration (duration). This dataset serves as the primary reference for tracking trial history, evaluating convergence behavior, and identifying the optimal hyperparameter configurations selected to maximize the model's predictive accuracy.

###############
data/processed/splits/test.csv
This dataset (test.csv) serves as the evaluation split for the machine learning model, containing network traffic session records primarily focused on port 8080 via the TCP protocol (Protocol 6). Each entry tracks an individual network session identified by a unique session_id, along with its precise timestamp and flow_duration measured in microseconds. This structured flow data is utilized to test and validate the model's performance in classifying network behaviors and detecting potential ransomware activities.

###############
data/processed/splits/train.csv
The train.csv file represents the processed training split for the machine learning model within the ransomware_ml pipeline. Located in the data/processed/splits/ directory, this dataset contains cleaned and engineered network session features—such as session_id and dst_port—extracted from network traffic logs. It is utilized to train the predictive model to distinguish between normal activities and potential ransomware behaviors before evaluation on the validation and test splits.

###############
data/processed/splits/val.csv
The val.csv file acts as the critical benchmark during model development, serving as the validation split for the ransomware detection pipeline. Containing high-dimensional, time-series network flow statistics—such as packet lengths, packet frequencies, and throughput metrics (Flow Byts/s, Flow Pkts/s)—this dataset is explicitly isolated from the training process. Its primary function is to provide an unbiased evaluation of model performance during hyperparameter tuning and optimization phases. By evaluating network traffic behaviors against this unseen data, the pipeline effectively mitigates overfitting, ensures robust generalization to novel connection patterns, and guarantees that the final classification model can reliably identify ransomware anomalies before deployment.
###############

data/processed/ransomware_impact_dataset.csv
This dataset contains network traffic session logs designed for analyzing and detecting ransomware behavior and its impact on a network. Each record captures a discrete network event, tracking essential connection attributes such as the unique session_id, the destination port (dst_port), and the transport layer protocol (represented numerically, e.g., 6 for TCP and 17 for UDP). Additionally, every session is precisely logged with a timestamp to enable chronological sequencing and time-series analysis of network anomalies. This structured data provides a foundation for training machine learning models to identify patterns indicative of ransomware propagation or command-and-control communication.


📌 Project Documentation Note

The detailed descriptions above document the purpose, workflow, datasets, reports, figures, and model artifacts included in this project. The original project descriptions have been retained in full.

🧪 Reproducibility

Project configuration is centralised through config.yaml, while the documented preprocessing, feature-engineering, training, optimization, evaluation, and prediction stages provide a structured workflow for reproducing the ransomware economic impact prediction pipeline.

🧠 Explainability

The project incorporates SHAP-based explainability and sensitivity analysis to support interpretation of model behaviour and to assess the contribution of sector-related features to economic impact prediction.

🚀 Deployment

The trained model can be exposed as a live prediction service through the FastAPI layer described in src/api.py, while src/predict.py provides an end-to-end inference workflow and command-line interface.
