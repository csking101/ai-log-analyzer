# **Pre-Application Design Document: AI-Powered Log Triage & Security Alert Aggregator for Fedora**

## **1. Project Overview**
This project aims to **automatically parse, classify, and prioritize security-related logs** on a Fedora system. The tool will aggregate logs from multiple sources (e.g., SELinux, systemd journal, audit logs) and apply **Machine Learning (ML) and Natural Language Processing (NLP)** techniques to identify and prioritize potential security events. The goal is to help administrators quickly detect and respond to critical alerts while **filtering out routine noise**.

---

## **2. Design Considerations (AI-Focused)**

### **2.1 Log Aggregation & Preprocessing**
- Logs will be collected from **SELinux, systemd, audit logs, SSH logs, and firewall logs**.
- Preprocessing includes **removing duplicates, extracting structured entities (IP, Usernames, Timestamps), and filtering routine messages** using rule-based heuristics.
- **Log Template Extraction (Drain3)** will be used to cluster similar logs and reduce redundancy.

### **2.2 Named Entity Recognition (NER) for Security Logs**
- Extract **key security entities** such as **usernames, IP addresses, file paths, processes**.
- Use a **hybrid approach**:
  - **Regex for structured data** (e.g., extracting IPs, timestamps).
  - **SpaCy for general NER** (e.g., usernames, processes).
  - **BERT-based NER for contextual analysis** (e.g., attack patterns like SQL Injection).

### **2.3 Feature Engineering for ML Models**
- Convert logs into numerical representations for anomaly detection:
  - **TF-IDF for simple keyword-based feature extraction**.
  - **Word2Vec or BERT embeddings for semantic log understanding**.
  - **One-hot encoding or frequency encoding for categorical entities (Users, IPs, Process Names).**

### **2.4 Anomaly Detection**
We need a **fast, lightweight anomaly detection model** to flag suspicious logs **before sending them to an LLM for deeper analysis**. The following models will be tested:

- **Isolation Forest** (Detects rare log patterns that deviate from normal activity.)
- **One-Class SVM** (Finds anomalies based on distance from a learned normal distribution.)
- **Autoencoder (Neural Network)** (Learns a compressed representation of normal logs; high reconstruction error indicates anomalies.)

Each model will be evaluated based on **False Positive Rate, Detection Accuracy, and Speed**.

### **2.5 LLM-Based Contextual Threat Analysis**
- If an anomaly is detected, it will be **sent to a quantized LLM** for deeper reasoning.
- **LLM Model Choices:**
  - **DistilBERT (Efficient for log classification & quick analysis).**
  - **LLaMA-2 (Quantized, for contextual threat evaluation).**
  - **Mistral-7B (For complex multi-log analysis).**
- LLM will be prompted with security logs and asked to classify or explain them.
- **Example Prompt:**
  ```
  Analyze this log entry: "Failed password for root from 192.168.1.10 multiple times."
  Does this indicate a potential security threat?
  ```

### **2.6 Alerting & Reporting**
- **Immediate Alerts:** If a critical security event is detected, an **email or Slack notification** will be triggered.
- **Visualization Dashboard:** Use **Grafana** to visualize real-time security logs.
- **Database Storage:** Store classified logs in **PostgreSQL or SQLite** for historical analysis.

---

## **3. Model Selection & Training Plan**

### **3.1 Do We Need a Custom Model?**
- **For Anomaly Detection:** Off-the-shelf models (**Isolation Forest, One-Class SVM, Autoencoder**) are sufficient and will be fine-tuned with Fedora-specific log data.
- **For NER:** Fine-tuning **BERT-based NER on security logs** will improve detection of custom log entities.
- **For LLM Analysis:** Using **pretrained, quantized LLMs (LLaMA-2, Mistral-7B)** instead of training from scratch.

### **3.2 Data Collection & Fine-Tuning Plan**
- **Training Data Sources:**
  - Real-world logs from Fedora (`journalctl`, `auditd`, `firewalld`).
  - Public security datasets (e.g., CICIDS Intrusion Detection Dataset, OSSEC logs).
  - Simulated attack scenarios (brute-force login attempts, port scans).

- **Fine-Tuning Process:**
  - **BERT-NER Model:** Fine-tune on security logs to improve entity recognition.
  - **Autoencoder / Anomaly Models:** Train on normal logs and detect deviations.
  - **LLM Adaptation:** Use **LoRA fine-tuning** on a small subset of security logs.

---

## **4. Final Workflow Summary**

1. **Log Collection** → Extract logs from Fedora’s security subsystems.  
1. **Preprocessing & Noise Reduction** → Apply regex filtering, Drain3, tokenization.  
1. **NER Extraction** → Identify IPs, users, process names, attack patterns.  
1. **Feature Engineering** → Convert logs into TF-IDF, embeddings, categorical encodings.  
1. **Anomaly Detection** → Use Isolation Forest / Autoencoder to flag suspicious logs.  
1. **LLM-Based Analysis** → Pass flagged logs to LLaMA-2 or Mistral-7B for reasoning.  
1. **Alerting & Reporting** → Send alerts via email/Slack and visualize in Grafana.  


---
