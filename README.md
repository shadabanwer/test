trigger: none

pool:
  name: AzureICCAgentPool

parameters:

- name: listOfAPIs
  type: object
  default:
  - key: "api-master-azuredeploy"
    params: >-
      -keyVaultName "$(KVName)"
      -serviceBusNamespace "$(serviceBusNamespaceName)"
      -serviceBusTopicName "$(ServiceBusTopicName)"
      -API_Provider_AppId "$(AppRegistrationAppId)"
      -API_Provider_Role "ICCWebhook.Writer"      
    apimApiName: "ICC-Events-Webhook"

- name: listOfStandardLogicApps
  type: object
  default:
  - key: "LogicApp-Upgrade-azuredeploy"
    stdlogicAppNameDir: "stdLogicAppUpgrade"
    params: >-
      -logicAppName "$(logicAppName)"
      -appServicePlanName "$(appServicePlanName)"
      -storageConnectionString "$(storageConnectionString)"
      -AlertCc "$(AlertCc)"
      -AlertFrom "$(AlertFrom)"
      -AlertTo "$(AlertTo)"
      -KVPathPROD "$(KVPathPROD)"
      -apimAlertURI "$(apimAlertURI)"
      -subscriptionID "$(subscriptionID)"
      -environmentName "$(environmentName)"
      -rgName "$(rgNameLA)"
      -workspaceID "$(workspaceID)"
      -subscriptionName "$(subscriptionName)"
      -AINConnectionString "$(AINConnectionString)"
      -AlertSubjectCertificate "$(AlertSubjectCertificate)"
      -Environment "$(Environment)"
      -apimAlertURLCertificate "$(apimAlertURLCertificate)"
      -ASEName "$(ASEName)"
      -alertSubjectAINv2 "$(alertSubjectAINv2)"
      -maxUpdate "$(maxUpdate)"
      -maxUpdateCert "$(maxUpdateCert)"
      -KVPath "$(KVPath)"
      -AINInstrumentationKey "$(AINInstrumentationKey)"
      -oldCertThumbprint1 "$(oldCertThumbprint1)"
      -oldCertThumbprint2 "$(oldCertThumbprint2)"
      -oldCertThumbprint3 "$(oldCertThumbprint3)"
      -newCertThumbprint "$(newCertThumbprint)"
      -iccWebsiteChangeEventsTopicName "$(ServiceBusTopicName)"
      -processWebsiteChangeEventsTopicSubscriptionName "$(EventsTopicSubscriptionName)"
      -processAppSettingUpdateActionsTopicSubscriptionName "$(ActionsTopicSubscriptionName)"
      -processAppSettingSendNotificationsTopicSubscriptionName "$(NotificationsTopicSubscriptionName)"
      -serviceBusResourceGroupName "$(resourceGroupCommon)"
      -serviceBusNamespace "$(serviceBusNamespaceName)"
      -KVName "$(KVName)"
      -AlertApiClientIdKey "$(KeyVaultAlertsEmailClientIdKey)"
      -AlertApiClientSecretKey "$(KeyVaultAlertsEmailClientSecretKey)"
      -AlertApiSubscriptionKeyKey "$(KeyVaultAlertsEmailSubscriptionKeyKey)"
      -storageConnectionStringKey "$(KeyVaultStorageConnectionStringKey)"
      -deploymentEnvironment "$(automatedAlertDeploymentEnvironment)"
      -actionGroupResourceGroup "$(resourceGroupCommon)"
      -actionGroupName "$(automatedAlertsActionGroupName)"
      -aspCpuThreshold "$(automatedAlertsAspCpuThreshold)"
      -aspMemoryPercentageThreshold "$(automatedAlertsAspMemoryPercentageThreshold)"
      -processServerfarmChangeEventsTopicSubscriptionName "$(ServerfarmChangeEventsTopicSubscriptionName)"
      -azureDevOpsUriRelativePath "$(azureDevOpsUriRelativePath)"
      -azureDevOpsBasicAuthValueKey "$(KeyVaultAzureDevOpsBasicAuthValueKey)"
      -processAppSettingEventsTopicSubscriptionName "$(AppSettingChangeEventsTopicSubscriptionName)"
      -disabledWorkflowAlertRunbookWebhook "$(disabledWorkflowAlertRunbookWebhook)"

- name: StandardLogicAppWorkflows
  type: object
  default:
  - key: "Upsert-LASettings-wf"
    stdlogicAppNameDir: "stdLogicAppUpgrade"
    appName: '$(logicAppName)'

- name: listOfServiceBusTopics
  type: object
  default:
  - key: "ServiceBusTopics"
    params: >-
      -serviceBusNamespaceName "$(serviceBusNamespaceName)"
      -serviceBusTopicName "$(ServiceBusTopicName)"
      -lockDuration "$(ServiceBusLockDuration)"
      -serviceBusEventsTopicSubscriptionName "$(EventsTopicSubscriptionName)"
      -serviceBusEventsTopicSubscriptionSqlFilter "$(EventsTopicSubscriptionSqlFilter)"
      -serviceBusActionsTopicSubscriptionName "$(ActionsTopicSubscriptionName)"
      -serviceBusActionsTopicSubscriptionSqlFilter "$(ActionsTopicSubscriptionSqlFilter)"
      -serviceBusNotificationsTopicSubscriptionName "$(NotificationsTopicSubscriptionName)"
      -serviceBusNotificationsTopicSubscriptionSqlFilter "$(NotificationsTopicSubscriptionSqlFilter)"
      -serviceBusServerFarmEventsTopicSubscriptionName "$(ServerfarmChangeEventsTopicSubscriptionName)"
      -serviceBusServerFarmEventsTopicSubscriptionSqlFilter "$(ServerfarmChangeEventsTopicSubscriptionSqlFilter)"
      -serviceBusAppSettingEventsTopicSubscriptionName "$(AppSettingChangeEventsTopicSubscriptionName)"
      -serviceBusAppSettingEventsTopicSubscriptionSqlFilter "$(AppSettingChangeEventsTopicSubscriptionSqlFilter)"

- name: listOfKeyVaults
  type: object
  default:
   - key: "keyVault"
     params: 
       -rgName "$(KVRGName)"
       -kvName "$(KVName)"
       -logicAppName "$(logicAppName)"
       -utilityAlertsEmailClientIdValue "$(utilityAlertsEmailClientId)"
       -utilityAlertsEmailClientSecretValue "$(utilityAlertsEmailClientSecret)"
       -utilityAlertsEmailSubscriptionKeyValue "$(utilityAlertsEmailSubscriptionKey)"
       -utilityAlertsEmailClientIdKey "$(KeyVaultAlertsEmailClientIdKey)"
       -utilityAlertsEmailClientSecretKey "$(KeyVaultAlertsEmailClientSecretKey)"
       -utilityAlertsEmailSubscriptionKeyKey "$(KeyVaultAlertsEmailSubscriptionKeyKey)"
       -storageConnectionStringKey "$(KeyVaultStorageConnectionStringKey)"
       -storageConnectionStringValue "$(storageConnectionString)"
       -azureDevOpsBasicAuthValueKey "$(KeyVaultAzureDevOpsBasicAuthValueKey)"
       -azureDevOpsPatTokenValue "$(azureDevOpsPatToken)"

variables:
- name: BuildConfiguration
  value: Release
- name: VSTS_ARM_REST_IGNORE_SSL_ERRORS 
  value: true
- name: System.Debug
  value: true
- name: rgName
  value: $(rgNameLA)
- name: rgNameConnection
  value: 'RT-SYD-NPE-SPN-ICC-DEVTEST-AzIntegration'
- name: API_Folder_Name
  value: upsertLASettings
- name: API_Name # to be deleted after testing completes.
  value: upsertLASettings

stages:
- stage: Build
  displayName: 'Build and Validate (Dev)'
  jobs:
  - job: ValidateDeployment_Dev
    variables:
    - group: CommonVariables_Dev
    - group: LogicAppSettingUpsert_NPE
    - group: ICCAutomatedAlerts_NPE
    steps:
    - template: 'azure-validation-task.yml'
      parameters:
        listOfServiceBusTopics: ${{ parameters.listOfServiceBusTopics }}
        listOfKeyVaults: ${{ parameters.listOfKeyVaults }}
        listOfAPIs: ${{ parameters.listOfAPIs }}
        listOfStandardLogicApps: ${{ parameters.listOfStandardLogicApps }}
        StandardLogicAppWorkflows: ${{ parameters.StandardLogicAppWorkflows }}
        rgNameConnection: 'RT-SYD-NPE-SPN-ICC-DEVTEST-AzIntegration'
        deploymentMode: 'Validation'

  - job: 'PublishTemplates'
    displayName: 'Publish Templates'
    dependsOn: ValidateDeployment_Dev
    steps:
    - task: CopyFiles@2
      displayName: 'Copy Files to: $(Build.ArtifactStagingDirectory)'
      inputs:
        SourceFolder: '$(Build.SourcesDirectory)'
        Contents: |
          $(API_Folder_Name)/**
          !Pipelines/*
          ICCPlatformModules/**
        TargetFolder: '$(Build.ArtifactStagingDirectory)'
    - task: PublishBuildArtifacts@1
      displayName: "Publish Artifacts"
      inputs:
        PathtoPublish: '$(Build.ArtifactStagingDirectory)'
        artifactName: 'drop'

- stage: 'Deploy_Dev'
  displayName: 'NPE-Dev Deploy'
  dependsOn: Build
  variables:
  - group: CommonVariables_Dev
  - group: LogicAppSettingUpsert_NPE
  - group: ICCAutomatedAlerts_NPE
  jobs:
  - deployment: DeploySolution
    displayName: 'Deploy Solution'
    environment: API-Support-DEV
    strategy: 
      runOnce:
        deploy:
          steps:
          - download: current
          - template: 'azure-deployment-task.yml'
            parameters:
              listOfServiceBusTopics: ${{ parameters.listOfServiceBusTopics }}
              listOfKeyVaults: ${{ parameters.listOfKeyVaults }}
              listOfAPIs: ${{ parameters.listOfAPIs }}
              listOfStandardLogicApps: ${{ parameters.listOfStandardLogicApps }}
              StandardLogicAppWorkflows: ${{ parameters.StandardLogicAppWorkflows }}
              rgNameConnection: 'RT-SYD-NPE-SPN-ICC-DEVTEST-AzIntegration'
              deploymentMode: 'Incremental'

- stage: 'Deploy_Prod'
  displayName: 'Prod Deploy'
  dependsOn: Deploy_Dev
  condition: and( succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/master') )
  variables:
  - group: CommonVariables_Prod
  - group: LogicAppSettingUpsert_PROD
  - group: ICCAutomatedAlerts_Prod
  jobs:
  - deployment: DeploySolution
    displayName: 'Deploy Solution'
    environment: API-Support-Release
    strategy: 
      runOnce:
        deploy:
          steps:
          - download: current
          - template: 'azure-deployment-task.yml'
            parameters:
              listOfServiceBusTopics: ${{ parameters.listOfServiceBusTopics }}
              listOfKeyVaults: ${{ parameters.listOfKeyVaults }}
              listOfAPIs: ${{ parameters.listOfAPIs }}
              listOfStandardLogicApps: ${{ parameters.listOfStandardLogicApps }}
              StandardLogicAppWorkflows: ${{ parameters.StandardLogicAppWorkflows }}
              rgNameConnection: 'RT-SYD-PRD-SPN-ICC-PROD-AzIntegration'
              deploymentMode: 'Incremental'

- stage: 'Deploy_DR'
  displayName: 'DR Deploy'
  dependsOn: Deploy_Prod
  condition: and( succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/master') )
  variables:
  - group: CommonVariables_DR
  - group: LogicAppSettingUpsert_DR
  - group: ICCAutomatedAlerts_DR
  jobs:
  - deployment: DeploySolution
    displayName: 'Deploy Solution'
    environment: API-Support-Release
    strategy: 
      runOnce:
        deploy:
          steps:
          - download: current
          - template: 'azure-deployment-task.yml'
            parameters:
              listOfServiceBusTopics: ${{ parameters.listOfServiceBusTopics }}
              listOfKeyVaults: ${{ parameters.listOfKeyVaults }}
              listOfAPIs: ${{ parameters.listOfAPIs }}
              listOfStandardLogicApps: ${{ parameters.listOfStandardLogicApps }}
              StandardLogicAppWorkflows: ${{ parameters.StandardLogicAppWorkflows }}
              rgNameConnection: 'RT-TOR-PRD-SPN-ICC-DR-AzIntegration'
              deploymentMode: 'Incremental'

---------deployment-task
parameters:
  - name: listOfServiceBusTopics
    type: object
    default: []
  - name: listOfKeyVaults
    type: object
    default: []
  - name: listOfAPIs
    type: object
    default: []
  - name: listOfStandardLogicApps
    type: object
    default: []
  - name: StandardLogicAppWorkflows
    type: object
    default: []
  - name: rgNameConnection
    type: string
    default: ''
  - name: deploymentMode 
    type: string
    default: 'Incremental'

steps:
- script: echo $(rgNameConnection)
- ${{ each value in parameters.listOfServiceBusTopics }}:
  - template: '..\..\TemplateLibrary\PipelineTasks\servicebusTopicDeployment.yml'
    parameters:
     name: ${{ value.key }}
     filename: ${{ value.key }}-azuredeploy
     params: ${{ value.params }}
- ${{ each value in parameters.listOfKeyVaults }}:
  - template: '..\..\TemplateLibrary\PipelineTasks\keyVaultDeployment.yml'
    parameters:
      name: ${{ value.key }}
      filename: ${{ value.key }}-azuredeploy
      params: ${{ value.params }}
- ${{ each value in parameters.listOfAPIs }}:
  - template: '..\..\Template\Pipelines\apimDeploymentBicepV3.yml'
    parameters:
     name: API_${{ value.key }}
     filename: ${{ value.key }}
     params: ${{ value.params }}
     apimApiName: ${{ value.apimApiName }}
     serviceConnectionName: '$(rgNameConnection)'
     deploymentMode: 'Incremental'
- ${{ each value in parameters.listOfStandardLogicApps }}:
  - template: '..\..\Template\Pipelines\standardlogicAppDeploymentV1.1.yml'
    parameters:
      name: standardlogicApp_${{ value.key }}
      filename: ${{ value.key }}
      params: ${{ value.params }}
      stdlogicAppNameDir: ${{ value.stdlogicAppNameDir }}
#- script: tree $(Pipeline.Workspace) /f
- ${{ each value in parameters.StandardLogicAppWorkflows }}:
  - template: '..\..\Template\Pipelines\workflowDeploymentV1.1.yml'
    parameters:
      name: workflow_${{ value.key }}
      filename: ${{ value.key }}
      appname: ${{ value.appname }}
      stdlogicAppNameDir: ${{ value.stdlogicAppNameDir }}
