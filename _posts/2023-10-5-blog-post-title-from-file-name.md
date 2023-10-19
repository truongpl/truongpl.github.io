## Performance monitoring

Machine learning (ML) monitoring is crucial for maintaining the performance and reliability of ML systems. It serves several vital purposes:

- Quality Assurance: ML models can degrade over time due to changes in data distributions or the environment. Monitoring helps identify performance drops, ensuring that the model continues to provide accurate and reliable predictions.
- Data Quality: ML models are sensitive to changes in input data. Monitoring helps detect data drift and concept drift, allowing organizations to adapt their models to evolving conditions.
- Model Fairness: Monitoring can highlight bias and fairness issues in ML models, ensuring that predictions do not discriminate against specific demographic groups or exhibit harmful biases.
- Resource Management: Efficient resource utilization is critical. Monitoring can identify resource-intensive models and instances, helping organizations optimize their infrastructure.

In summary, ML monitoring is essential for maintaining the effectiveness, and compliance of ML systems while also optimizing resource usage and improving the user experience. It is a crucial practice for ensuring the stability of machine learning applications.

In this post I will try to cover the dashboard implementation, it is currently in work. My dashboard's design will look like:

![Dashboard_design](https://github.com/truongpl/truongpl.github.io/raw/main/docs/assets/Dashboard_design.png)

### Component description
- ML Service: already deployed on cloud
- PostgresDB: I used this database to store the metadata of system: users, api key, storage url, image url...
- Monitor model job: after some research, I decide to use Airflow, all I need is define and implement the DAG + DB task to get result + Scikit learn code to calculate ML metrics
- Dashboard: I have some experiences with Dash, using it will simplify the UI works.

