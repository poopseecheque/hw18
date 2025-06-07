# Week 6: MLOps. Sample Airflow Pipeline

## Agenda: 
1. How to access Apache Airflow.
2. Explore Airflow main components.
3. Build sample DAG.
4. Build DAG for data preprocessing.
5. Build DAG for models training.

## 1. How to access Apache Airflow.
1. You can access Airflow via gsom servers. For this you need to select airflow environment and explore ```__MANUAL/15_Airflow Demo.ipynb``` file.

2. Go to ```airflow/airflow.cfg``` file, update ```dags_folder``` field to the convenient for you (e.g. ```E2EML/Lab06```), save file.

3. In the terminal write ```start-ariflow.sh```

4. Go to the airflow UI (you got url for it in the 1st step)


## 2. Explore Airflow main components.
### 2.1 Dag
A DAG in Airflow represents a workflow as a collection of tasks with dependencies defined between them. It ensures tasks are executed in a specific order without cycles.
### 2.2 Operators
Operators define individual tasks in a workflow. They determine what actually gets done.
#### Bash operator
Executes bash commands. Useful for running scripts or command-line utilities.
#### Python Operator
Executes Python functions. Ideal for custom Python code execution within a workflow.
### 2.3 XCom
XCom (Cross-Communication) allows tasks to exchange messages or small amounts of data. It enables tasks to share information, facilitating more complex workflows.


## 3. Build sample DAG.
As a first, simple example, let's build a sample DAG to demonstrate how tasks can be chained together in Apache Airflow.

### Explanation:
#### read_env Task:
```python
def read_env():
    # Retrieve the data directory path from Airflow Variables
    data_dir = Variable.get("DATA_DIR")
    print(f"DATA DIR: {data_dir}")
    return data_dir
```

Retrieves the data directory path from Airflow Variables (need to be setuped from the Airflow UI -> Admin -> Variables).
Uses Variable.get("DATA_DIR") to fetch the path and prints it. 

Returns the data directory path for use in subsequent tasks via XCom.

#### compute_pi Task:
```python
def compute_pi():
    N = 10_000
    points_in_circle = 0 

    for i in range(N): 
        x, y = random.random(), random.random()

        if y <= (1 - x*x) ** 0.5: 
            points_in_circle += 1

    pi = 4 * points_in_circle / N
    print(f"Computed pi: {pi}")
    return pi
```
Computes an approximation of pi using a Monte Carlo method.
Generates random points within a unit square and checks if they lie within the unit circle. The ratio of points inside the circle to the total points is used to approximate pi.

Returns the computed value of pi.

#### compute_area Task:
```python
def compute_area(data_path, computed_pi, **kwargs): 
    circles = pd.read_csv(data_path+"/circles_data.csv")
    circles['approx_area'] = circles['radius'] * circles['radius'] * computed_pi
    circles['true_area'] = circles['radius'] * circles['radius'] * math.pi

    circles.to_csv(data_path+"/circle_areas.csv")
    print(circles)
```

Computes the approximate and true areas of circles using the computed pi.
Loads circle data from a CSV file, calculates the approximate and true areas, and saves the results to a new CSV file. Uses the data directory path from read_env and the computed pi from compute_pi.

Saves the results to a CSV file and prints the DataFrame. 

### Task Dependencies
```python
dag = DAG(dag_id="first-dag")

task_read_env = PythonOperator(
    task_id="read_env",
    python_callable=read_env,
    dag=dag,
    provide_context=True, 
)

task_compute_pi = PythonOperator(
    task_id="compute_pi",
    python_callable=compute_pi,
    dag=dag,
    provide_context=True
)

task_compute_area = PythonOperator(
    task_id="compute_area",
    python_callable=compute_area,
    dag=dag,
    provide_context=True,
    op_args=[task_read_env.output, task_compute_pi.output]
)

task_read_env >> task_compute_pi >> task_compute_area
```
The tasks are chained together using the >> operator, ensuring they execute in the order: read_env, compute_pi, compute_area.


## 4. Build DAG for data preprocessing.
![Data Preparation DAG](data_preprocessing_dag.png)

## 5. Build DAG for models training.
![Model training DAG](model_training_dag.png)

## 6. Project tasks. 
+ Download data from the source (url assigned to you)
+ Build data preprocessing pipeline in Airflow
    + Use full dataset
    + Include steps that you used in data preprocessing stage (week 3): 
        - Imputation
        - Feature extraction
        - Scaling
        
        Check that you fit all models only on the training part of data. Keep preprocessed data in the appropriate place in your repository. Save preprocessing pipeline, it should be able to perform all actions you do with dataset: scaling, imputing, feature extraction, data cleaning etc (all steps from week 3). 

+ Build model training pipeline. Pipeline must use models (and parameters) you used in model training stage (week 4), apply hyperparameter optimization techniques. Enshure that results that you achieved in week 4 report are close to results you get from training in Airflow. 
+ Make model pipeline save all trained models in ```models/``` directory. 
Also, pipeline should create ```models/best``` directory, find best model & combine with preprocessor (and save it in ```models/best```), log metrics of best model (e.g. in json/pdf format), plot appropriate charts describing your model performance. 
