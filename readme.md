 Project Overview

Traditional Network Intrusion Detection Systems (NIDS) rely on static blacklists and manual rule updates — approaches that are fundamentally reactive and unable to detect zero-day phishing attacks. This project builds a  MLOps pipeline that automatically classifies URLs as phishing or legitimate using machine learning, and keeps itself updated as traffic patterns evolve.


 📂 Dataset

UCI Phishing Websites Dataset
| Property | Details |
|----------|---------|
| **Source** | UCI Machine Learning Repository |
| **Citation** | Mohammad, R. M., Thabtah, F., & McCluskey, T. L. (2015). Phishing Websites Dataset. UCI ML Repository. https://archive.ics.uci.edu/ml/datasets/phishing+websites |
| **Records** | 11,055 URLs |
| **Features** | 30 engineered features + 1 target column (Result) |
| **Target** | -1 = Phishing, +1 = Legitimate (remapped to 0/1 during training) |
| **Storage** | Hosted on MongoDB Atlas, ingested programmatically at pipeline runtime |
| **Local copy** | `Network_Data/phisingData.csv` |

 Feature Categories

| Category | Features |
|----------|---------|
| **URL Structure** | `having_IP_Address`, `URL_Length`, `Shortining_Service`, `having_At_Symbol`, `double_slash_redirecting`, `Prefix_Suffix`, `having_Sub_Domain` |
| **SSL & Domain** | `SSLfinal_State`, `Domain_registeration_length`, `HTTPS_token`, `age_of_domain`, `DNSRecord` |
| **Page Content** | `Favicon`, `Request_URL`, `URL_of_Anchor`, `Links_in_tags`, `SFH`, `Submitting_to_email`, `Iframe`, `RightClick`, `popUpWidnow`, `on_mouseover` |
| **Traffic & Reputation** | `web_traffic`, `Page_Rank`, `Google_Index`, `Links_pointing_to_page`, `Statistical_report`, `Abnormal_URL`, `Redirect`, `port` |
| **Target** | `Result` |

All feature values are encoded as `-1` (phishing indicator), `0` (suspicious/neutral), or `1` (legitimate indicator).

---
 Pipeline Architecture

```
MongoDB Atlas
     │
     ▼
┌─────────────────────┐
│   Data Ingestion    │  → Feature Store CSV → train.csv + test.csv (80/20 split)
└─────────────────────┘
     │
     ▼
┌─────────────────────┐
│   Data Validation   │  → Schema check (31 columns) + KS-test drift report
└─────────────────────┘
     │
     ▼
┌─────────────────────┐
│ Data Transformation │  → KNN Imputer (k=3) → train.npy + test.npy + preprocessing.pkl
└─────────────────────┘
     │
     ▼
┌─────────────────────┐
│   Model Training    │  → 5-model grid search → best model → MLflow logging
└─────────────────────┘
     │
     ▼
┌─────────────────────┐
│   Flask REST API    │  → /predict (batch CSV) + /train (trigger retraining)
└─────────────────────┘
     │
     ▼
┌─────────────────────┐
│  Docker → AWS ECR   │  → GitHub Actions CI/CD → EC2 deployment
└─────────────────────┘
```

---

🧪 Models Evaluated

| Model | Notes |
|-------|-------|
| Random Forest  | Best performer — selected automatically by pipeline |
| Gradient Boosting | Sequential boosting, strong on tabular data |
| AdaBoost | Lightweight boosting, fast training |
| Decision Tree | Interpretable single-tree baseline |
| Logistic Regression | Linear baseline — lower bound for comparison |

All models are tuned via grid search cross-validation. The best model is selected automatically based on test-set F1-score and saved to `final_model/model.pkl`.




>**Please check the report uploaded on the google drive for additional information
**
