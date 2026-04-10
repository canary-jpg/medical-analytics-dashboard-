# Medical analytics dashboard

A 30-day readmission analysis built on 5,837 synthetic patients using Synthea, dbt, and DuckDB.

```sql overview
select
    count(*) as total_patients,
    sum(case when is_deceased then 1 else 0 end) as deceased_patients,
    sum(case when is_high_risk then 1 else 0 end) as high_risk_patients,
    round(avg(age_years), 1) as avg_age,
    round(avg(total_encounter_cost), 0) as avg_total_cost
from mart_patient_summary
```

<BigValue
    data={overview}
    value="total_patients"
    title="Total patients"
/>
<BigValue
    data={overview}
    value="high_risk_patients"
    title="High risk patients"
/>
<BigValue
    data={overview}
    value="avg_age"
    title="Avg age"
/>
<BigValue
    data={overview}
    value="avg_total_cost"
    title="Avg total cost"
    fmt="usd0"
/>

---

## Readmission overview
```sql readmissions
select
    count(*) as total_inpatients_encounters,
    sum(case when is_readmitted_30_day then 1 else 0 end) as total_readmissions,
    round(100.0 * sum(case when is_readmitted_30_day then 1 else 0 end) / count(*), 2) as readmission_rate_pct,
    round(avg(case when is_readmitted_30_day then total_claim_cost end), 0) as avg_cost_readmitted,
    round(avg(case when not is_readmitted_30_day then total_claim_cost end), 0) as avg_cost_not_readmitted
from mart_readmissions
```

<BigValue
    data={readmissions}
    value="total_inpatients_encounters"
    title="Inpatient encounters"
/>
<BigValue
    data={readmissions}
    value="total_readmissions"
    title="30-day readmissions"
/>
<BigValue
    data={readmissions}
    value="readmission_rate_pct"
    title="Readmission rate"
    fmt="pct1"
/>

---

## Readmission rate by condition

```sql by_condition
select
    reason_description as condition,
    count(*) as total_encounters,
    sum(case when is_readmitted_30_day then 1 end) as readmissions,
    round(100.0 * sum(case when is_readmitted_30_day then 1 else 0 end) / count(*), 1) as readmission_rate_pct
from mart_readmissions
where reason_description is not null
group by reason_description 
having count(*) >= 5
order by readmission_rate_pct desc
limit 10
```

<BarChart
    data={by_condition}
    x="condition"
    y="readmission_rate_pct"
    swapXY=true
    title="Top 10 conditions by readmission rate"
    yAxisTitle="Readmission rate (%)"
/>

---

## High risk vs low risk patients
```sql risk_profile
select
    case when is_high_risk then 'High Risk' else 'Low Risk' end as risk_group,
    count(*) as patients,
    round(avg(age_years), 1) as avg_age,
    round(avg(active_conditions), 1) as avg_active_conditions,
    round(avg(active_medications), 1) as avg_active_medications,
    round(avg(total_encounter_cost), 0) as avg_total_cost
from mart_patient_summary
group by is_high_risk
order by is_high_risk desc
```

<DataTable data={risk_profile}    />

--- 

## Cost comparison

```sql cost_comparison
select
    case when is_readmitted_30_day then 'Readmitted' else 'Not readmitted' end as readmission_status,
    count(*) as encounters,
    round(avg(total_claim_cost), 0) as avg_cost
from mart_readmissions
group by is_readmitted_30_day
```

<BarChart
    data={cost_comparison}
    x="readmission_status"
    y="avg_cost"
    title="Average encounter cost by readmission status"
/>