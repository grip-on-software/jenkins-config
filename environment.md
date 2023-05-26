The following environment variables should be configured on Jenkins in order to 
use the Declarative Pipeline multibranch jobs. Some may also be described 
further in their respective repositories:

- `ANALYSIS_ORGANIZATION`: Name of organization to retrieve data for by default
- `BIGBOAT_STATUS_CRON`: Cron-style string of interval at which to trigger the 
  BigBoat status visualization job
- `DOCKER_REGISTRY`: FQDN with optional port specification of the host where 
  Docker images can be pushed/pulled at
- `DOCKER_REPOSITORY`: FQDN without port of the host where Docker images can be 
  pushed/pulled at
- `DOCKER_URL`: URL of the host where Docker images can be pushed/pulled at
- `EXCHANGE_CRON`: Cron-style string of interval at which to trigger the export 
  exchange job
- `NPM_REGISTRY`: URL to the Node.js package registry where packages can be 
  downloaded from
- `PIP_REGISTRY`: URL to the PyPI registry where packages can be 
  published/retrieved at
- `PREDICTION_CRON`: Cron-style string of interval at which to trigger the 
  prediction jobs
- `PREDICTION_ORGANIZATIONS`: Space-separated list of organizations to collect
  prediction features for by default
- `PREDICTOR_REMOTE`: Boolean whether to perform predictions on a specifically 
  labeled worker node instead of the current worker
- `REPORT_PARAMS`: Additional site-wide parameters to provide to analysis 
  report scripts
- `TENSORFLOW_VERSION`: Docker tag to use for the prediction jobs as a base of 
  the Docker image from tensorflow. Include `-gpu` to use a GPU-enabled image
- `VISUALIZATION_ANONYMIZED`: Boolean whether to indicate that the 
  visualizations have anonymized data on the visualization site
- `VISUALIZATION_CRON`: Cron-style string of interval at which to trigger the 
  visualizations jobs
- `VISUALIZATION_MAX_SECONDS`: Number of seconds to wait for the visualizations 
  to finish building during the visualization site integration test
- `VISUALIZATION_ORGANIZATION`: Name of organization to build visualizations 
  for by default
