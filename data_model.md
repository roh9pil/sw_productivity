erDiagram

    %% --- Subject Area 1: Planning & Requirements ---
    project {
        int id PK "Project ID"
        varchar name "Project Name"
        varchar project_key "Project Key"
    }
    sprint {
        int id PK "Sprint ID"
        varchar name "Sprint Name"
        text goal "Sprint Goal"
        date start_date "Start Date"
        date end_date "End Date"
        varchar status "Status"
    }
    release_version {
        int id PK "Release ID"
        varchar version_name "Version Name"
        date release_date "Release Date"
        varchar status "Status"
    }
    team_member {
        int id PK "Member ID"
        varchar name "Member Name"
        varchar email "Email"
        int team_id "Team ID"
    }
    work_item {
        int id PK "Work Item ID"
        int project_id FK "Project ID"
        int sprint_id FK "Sprint ID"
        int release_version_id FK "Release ID"
        int epic_id FK "Epic ID"
        varchar type "Type (Story, Bug, Task)"
        varchar status "Status"
        varchar priority "Priority"
        varchar summary "Summary"
        int assignee_id FK "Assignee ID"
        int reporter_id FK "Reporter ID"
        datetime created_at "Created Timestamp"
        datetime updated_at "Updated Timestamp"
        datetime resolved_at "Resolved Timestamp"
        float story_points "Story Points"
    }

    %% --- Subject Area 2: Code & Development ---
    repository {
        int id PK "Repo ID"
        varchar name "Repo Name"
        varchar owner_name "Owner Name"
    }
    commit {
        varchar commit_hash PK "Commit Hash"
        int repository_id FK "Repo ID"
        int author_id FK "Author ID"
        datetime committed_at "Commit Timestamp"
        text message "Commit Message"
    }
    file_change {
        int id PK "Change ID"
        varchar commit_hash FK "Commit Hash"
        varchar file_path "File Path"
        int lines_added "Lines Added"
        int lines_deleted "Lines Deleted"
    }
    pull_request {
        int id PK "PR ID"
        int repository_id FK "Repo ID"
        int author_id FK "Author ID"
        varchar status "Status"
        varchar title "Title"
        datetime created_at "Created Timestamp"
        datetime merged_at "Merged Timestamp"
        datetime closed_at "Closed Timestamp"
    }
    code_review {
        int id PK "Review ID"
        int pull_request_id FK "PR ID"
        int reviewer_id FK "Reviewer ID"
        varchar status "Status (Approved, etc.)"
        datetime submitted_at "Submitted Timestamp"
    }

    %% --- Subject Area 3: Build & Deployment ---
    pipeline_run {
        int id PK "Run ID"
        int repository_id FK "Repo ID"
        varchar trigger_commit_hash FK "Trigger Commit Hash"
        varchar status "Status (Success, Failure)"
        datetime started_at "Start Timestamp"
        datetime finished_at "End Timestamp"
        int duration_seconds "Duration in Seconds"
    }
    deployment {
        int id PK "Deployment ID"
        int pipeline_run_id FK "Pipeline Run ID"
        varchar environment_name "Environment"
        datetime deployed_at "Deployment Timestamp"
        varchar status "Status (Success, Failure)"
    }
    build_artifact {
        int id PK "Artifact ID"
        int pipeline_run_id FK "Pipeline Run ID"
        varchar name "Artifact Name"
        varchar version "Artifact Version"
    }

    %% --- Subject Area 4: Quality & Testing ---
    test_case {
        int id PK "Test Case ID"
        int work_item_id FK "Work Item ID"
        varchar title "Title"
        varchar priority "Priority"
    }
    test_run {
        int id PK "Test Run ID"
        int test_case_id FK "Test Case ID"
        int deployment_id FK "Deployment ID"
        datetime executed_at "Execution Timestamp"
        varchar status "Status (Pass, Fail)"
        int duration_seconds "Duration in Seconds"
    }
    defect {
        int id PK "Defect ID (FK to work_item.id)"
        varchar severity "Severity"
        int found_in_release_id FK "Found in Release ID"
        text root_cause_description "Root Cause"
    }
    code_quality_scan {
        int id PK "Scan ID"
        varchar commit_hash FK "Commit Hash"
        float test_coverage_percentage "Test Coverage (%)"
        int cyclomatic_complexity "Cyclomatic Complexity"
        float duplication_percentage "Duplication (%)"
        int vulnerability_count "Vulnerability Count"
    }

    %% --- Subject Area 5: Operations & Reliability ---
    service {
        int id PK "Service ID"
        varchar name "Service Name"
        int owner_team_id "Owner Team ID"
    }
    incident {
        int id PK "Incident ID"
        int service_id FK "Service ID"
        varchar severity "Severity"
        varchar status "Status"
        datetime started_at "Start Timestamp"
        datetime resolved_at "Resolved Timestamp"
        int root_cause_deployment_id FK "Root Cause Deployment ID"
    }
    sli_measurement {
        datetime measured_at PK "Measurement Timestamp"
        int service_id PK "Service ID"
        varchar metric_name "Metric Name"
        float metric_value "Metric Value"
    }
    service_level_objective {
        int id PK "SLO ID"
        int service_id FK "Service ID"
        varchar sli_metric_name "SLI Metric Name"
        float target_value "Target Value"
        int time_window_days "Time Window (Days)"
    }

    %% --- Relationships ---
    project |

|--o{ work_item : contains
    sprint |

|--o{ work_item : contains
    release_version |

|--o{ work_item : contains
    team_member |

|--o{ work_item : "assignee"
    team_member |

|--o{ work_item : "reporter"
    work_item |

|--|
| defect : "can be"

    repository |

|--o{ commit : contains
    team_member |

|--o{ commit : "author"
    commit |

|--o{ file_change : contains
    repository |

|--o{ pull_request : contains
    team_member |

|--o{ pull_request : "author"
    pull_request |

|--o{ code_review : has
    team_member |

|--o{ code_review : "reviewer"

    repository |

|--o{ pipeline_run : "belongs to"
    commit }o--|

| pipeline_run : "triggers"
    pipeline_run |

|--o{ deployment : "results in"
    pipeline_run |

|--o{ build_artifact : "produces"

    work_item }o--|

| test_case : "is covered by"
    test_case |

|--o{ test_run : "has"
    deployment }o--|

| test_run : "validates"
    release_version }o--|

| defect : "found in"
    commit }o--|

| code_quality_scan : "is scanned"

    service |

|--o{ incident : "affects"
    deployment }o--|

| incident : "can cause"
    service |

|--o{ sli_measurement : "has"
    service |

|--o{ service_level_objective : "has"

