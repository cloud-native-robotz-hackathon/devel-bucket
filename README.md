# Development 🪣


![overview.drawio.v2.png](overview.drawio.v2.png)

## Workshop flow

 * By default be build the entire end-to-end solution
 * Depend on the audience we remove parts like:
     * AI/ML Model Training
     * Application development (starter-app-python)
     * Maybe development in different languages because it just consume and ML/API, CAM IP and Robot API
     * Build GitOps infrastructure and rollout applications
     * Pipeline for all applications (Maybe pipeline as code )
     * …
 * The flow is:
     * Development Application in DataCenter - or just test / with Robot Access
     * Train model with new Objects
     * Close connection to robot
     * Depoy application on Robot
     * Let’s drive the Robot in a competition situation against other teams.


## Open Issues

- [ ] Containerize GoPiGo3
      https://github.com/cloud-native-robotz-hackathon/edgecontroller/issues/1
