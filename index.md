## Databricks workspace using Microsoft CAF

# Setup
Detailed Reference Guide : [Getting started](https://github.com/Azure/caf-terraform-landingzones/blob/master/documentation/getting_started/getting_started.md)
1. Check if you have vscode requirements ready i.e  Visual Studio Code Extension - Remote Development
    Reference:https://github.com/aztfmod/rover#pre-requisites
2. Clone CAF databricks starter template [CAF_DATABRICKS_STARTER](https://github.com/lordlinus/caf-databricks-with-kv).   
    Optional: git clone --branch 2101.0.0 https://github.com/Azure/caf-terraform-landingzones.git /tf/caf/public. 
3. Open in vs-code container
4. Check if you have owner privileges on your account if you see the below error.   
	`Error on or near line 182: the current account must have Owner privilege on the subscription to deploy launchpad.; exiting with status 2. Ensure you have owner persmissions`. 

# Rover login
```markdown
rover login
```


# Set Environement variable
```markdown
export environment=sandpit
export caf_env=demo
export base_landingzone_tfstate_name="databricks_workspace"
export db_cluster_type="101-simple-cluster"
```

# Deploy the launchpad
```markdown
rover -lz /tf/caf/public/landingzones/caf_launchpad \
  -var-folder /tf/caf/configuration/${environment}/level0/launchpad \
  -parallelism 30 \
  -level level0 \
  -env ${caf_env} \
  -launchpad \
  -a apply
```

# Deploy Level1
```markdown
rover -lz /tf/caf/public/landingzones/caf_foundations/ \
  -var-folder /tf/caf/configuration/${environment}/level1 \
  -parallelism 30 \
  -level level1 \
  -env ${caf_env} \
  -a apply
```

# Deploy Level2 shared services in pass through mode
```markdown
mkdir /tmp/empty

rover -lz /tf/caf/public/landingzones/caf_shared_services/ \
  -tfstate caf_shared_services.tfstate \
  -var-folder /tmp/empty \
  -parallelism 30 \
  -level level2 \
  -env ${caf_env} \
  -a apply
```

# Deploy Level2 networking with Hub-and-spoke design
```markdown
rover -lz /tf/caf/public/landingzones/caf_networking/ \
  -tfstate networking_hub.tfstate \
  -var-folder /tf/caf/configuration/${environment}/level2/networking/hub \
  -parallelism 30 \
  -level level2 \
  -env ${caf_env} \
  -a apply
```

# Deploy databricks workspace as Level3
```markdown
rover -lz /tf/caf/public/landingzones/caf_solutions/ \
  -var-folder /tf/caf/reference_implementations/data_analytics/databricks/${db_cluster_type}\
  -tfstate ${base_landingzone_tfstate_name}.tfstate \
  -env ${caf_env} \
  -level level3 \
  -a apply
```

# Deploy databricks cluster
```markdown
rover -lz /tf/caf/public/landingzones/caf_solutions/add-ons/databricks \
  -var-folder /tf/caf/reference_implementations/data_analytics/databricks/${db_cluster_type}/cluster \
  -tfstate databricks_cluster.tfstate \
  -var tfstate_key=${base_landingzone_tfstate_name}.tfstate \
  -env ${caf_env} \
  -level level3 \
  -a apply
```


Reference
1. Onboarding video: https://www.youtube.com/watch?v=M5BXm30IpdY
2. CAF Docs: https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/
3. Starter Template: https://github.com/Azure/caf-terraform-landingzones-starter
5. 
