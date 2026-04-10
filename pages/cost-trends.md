# Cost trends

Encounters cost broken down by care setting, payer coverage, and patient demographics.

```sql cost_by_class
select
    encounter_class,
    count(*) as encounters,
    round(avg(total_claim_cost), 0) as avg_cost,
    round(sum(total_claim_cost), 0) as total_cost
from mart_readmissions
group by encounter_class
order by avg_cost desc
```

<BarChart
    data={cost_by_class}
    x="encounter_class"
    y="avg_cost"
    title="Average encounter by care setting"
    yAxisTitle="Avg cost (USD)"
/>

---

## Cost by readmission status and care setting

```sql cost_by_class_readmission
select
    encounter_class,
    case when is_readmitted_30_day then 'Readmitted' else 'Not Readmitted' end as readmission_status,
    count(*) as encounters,
    round(avg(total_claim_cost), 0) as avg_cost
from mart_readmissions
group by encounter_class, is_readmitted_30_day
order by encounter_class, is_readmitted_30_day desc
```

<BarChart
    data={cost_by_class_readmission}
    x="encounter_class"
    y="avg_cost"
    series="readmission_status"
    title="Avg cost by care setting and readmission status"
    yAxisTitle="Avg cost (USD)"
    type="grouped"
/>

---

## Cost by Age group

```sql cost_by_age
select
    case 
        when age_years < 18 then 'Under 18'
        when age_years < 35 then '18-34'
        when age_years < 50 then '35-49'
        when age_years < 65 then '50-64'
        when age_years < 80 then '65-79'
        else '80+'
    end age_group,
    count(*) as patients,
    round(avg(total_encounter_cost), 0) as avg_encounter_cost,
    round(avg(total_medication_cost), 0) as avg_medication_cost,
    sum(case when is_high_risk then 1 else 0 end) as high_risk_patients
from mart_patient_summary
group by age_group
order by min(age_years)    
```

<BarChart
    data={cost_by_age}
    x="age_group"
    y="avg_encounter_cost"
    title="Average encounter cost by age group"
    yAxisTitle="Avg cost (USD)"
/>

<DataTable data={cost_by_age} />