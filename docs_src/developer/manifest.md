# Manifest File (manifest.xml)

The manifest is the **deployment blueprint** for each AI Service. It centralizes all deployment metadata so that any change — new version, different sizing, extra databases — happens in one file and is versioned with the code.

## Structure

```xml
<Package>
  <ArtifactInfo>
    <Name>suspicious_activity_detection</Name>        <!-- Must match ZIP filename -->
    <DisplayName>Suspicious Activity Detection</DisplayName>
    <Description>Detects suspicious activity from person tracks</Description>
    <Module_Name>DataFactory</Module_Name>
    <Component>Preprocessing</Component>
    <AIServiceVersion>1.0</AIServiceVersion>
    <EnvironmentConfig>config/env.config</EnvironmentConfig>
    <InputPayLoad>config/input.json</InputPayLoad>
    <OutputResponse>config/output.json</OutputResponse>
    <Feedback>config/feedback.json</Feedback>
    <metainfo industry="Security" collection="Vision" />
  </ArtifactInfo>

  <Configuration>
    <Deployment type="Algo" technology="Azure"
                route="/datafactory/preprocessing/suspicious_activity"
                cluster="shared">
      <DevSize cpu="2" memory="4Gi" DR="no" />
      <QaSize cpu="4" memory="8Gi" DR="no" />
      <StagingSize cpu="8" memory="16Gi" DR="yes" />
      <ProdSize cpu="16" memory="32Gi" DR="yes" />
    </Deployment>

    <DBServices>
      <DBService type="SQL" provider="AzureSQL">
        <DevSize tier="Basic" />
        <ProdSize tier="GeneralPurpose" />
      </DBService>
    </DBServices>
  </Configuration>
</Package>
```

## Key Fields

| Field | Purpose |
|-------|---------|
| `Name` | Unique machine-readable ID. Must match the ZIP filename exactly. |
| `Module_Name` | Platform module (e.g., DataFactory). Visible in UI navigation. |
| `Component` | Sub-category (Ingestion, Preprocessing, Storage, etc.) |
| `route` | HTTP endpoint path registered in the API gateway |
| `type="Algo"` | Azure-deployed algorithmic compute. Use `type="Data"` for Databricks. |
| `cluster="shared"` | Use shared Kubernetes cluster. Use `"exclusive"` for dedicated compute. |
| `DevSize/QaSize/StagingSize/ProdSize` | VM/container resource allocation per environment |
| `DR="yes"` | Enable disaster-recovery replicas for staging/production |
