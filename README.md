# data-pipeline
created a simple data pipeline to minimise time on doing etc tasks
class DataPipelineManager:
    def __init__(self):
        self.pipeline_definitions = []
        self.human_intervention_queue = []
    
    def define_pipeline(self, source, transformations, destination, requires_human_intervention=False):
        pipeline = {
            "source": source,
            "transformations": transformations,
            "destination": destination,
            "requires_human_intervention": requires_human_intervention
        }
        self.pipeline_definitions.append(pipeline)
    
    def execute_pipeline(self, pipeline):
        data = pipeline["source"].extract()
        transformed_data = pipeline["transformations"].apply(data)
        if pipeline["requires_human_intervention"]:
            self.human_intervention_queue.append(transformed_data)
        else:
            pipeline["destination"].load(transformed_data)
    
    def monitor_pipelines(self):
        for pipeline in self.pipeline_definitions:
            self.execute_pipeline(pipeline)
    
    def intervene_human_tasks(self):
        for task in self.human_intervention_queue:
            # Notify user to review and take action on the task
            user_action = get_user_input("Review and approve/reject task: ", task)
            if user_action == "approve":
                pipeline = self.find_pipeline_for_task(task)
                pipeline["destination"].load(task)
            self.human_intervention_queue.remove(task)
    
    def find_pipeline_for_task(self, task):
        for pipeline in self.pipeline_definitions:
            if pipeline["transformations"].apply(task) == task:
                return pipeline

# Example usage:
pipeline_manager = DataPipelineManager()

# Define pipelines
pipeline_manager.define_pipeline(source=CSVDataSource("data.csv"),
                                 transformations=DataTransformations(),
                                 destination=DatabaseDestination("mydb"),
                                 requires_human_intervention=False)

pipeline_manager.define_pipeline(source=APISource("api.endpoint"),
                                 transformations=DataTransformations(),
                                 destination=DatabaseDestination("mydb"),
                                 requires_human_intervention=True)

# Monitor and execute pipelines
pipeline_manager.monitor_pipelines()

# Review and approve human intervention tasks
pipeline_manager.intervene_human_tasks()
