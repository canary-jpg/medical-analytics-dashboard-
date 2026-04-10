# Risk tier breakdown

Patient distribution across low, medium, and high readmission risk tiers.

```sql risk_tiers
select 
    case
        when readmission_rate_pct = 0 then 'Low'
        when readmission_rate_pct <= 50 then 'Medium'
        else 'High'
    end as risk_tier,
    count(*) as patients,
    round(avg(age_years), 1) as avg_age,
    round(avg(active_conditions), 1) as avg_active_conditions,
    round(avg(active_medications), 1) as avg_active_medications,
    round(avg(total_encounter_cost), 0) as avg_total_cost,
    round(avg(total_inpatient_encounters), 1) as avg_inpatient_encounters
from mart_patient_summary
group by risk_tier
order by 
    case risk_tier when 'High' then 1 when 'Medium' then 2 else 3 end
```

<BigValue
    data={risk_tiers}
    value="patients"
    title="Patients"
/>

<BarChart
    data={risk_tiers}
    x="risk_tier"
    y="patients"
    title="Patients by risk tier"
    yAxisTitle="Patients"
/>

---

## Profile by risk tier

<DataTable data={risk_tiers} />

---

## Cost by risk tier

```sql cost_by_tier
select
    case 
        when readmission_rate_pct = 0 then 'Low'
        when readmission_rate_pct <= 50 then 'Medium'
        else 'High'
    end as risk_tier,
    round(avg(total_encounter_cost), 0) as avg_encounter_cost,
    round(avg(total_medication_cost), 0) as avg_medication_cost,
    round(avg(total_encounter_cost + total_medication_cost), 0) as avg_total_cost
from mart_patient_summary
group by risk_tier
order by 
    case risk_tier when 'High' then 1 when 'Medium' then 2 else 3 end
```

<BarChart
    data={cost_by_tier}
    x="risk_tier"
    y={["avg_encounter_cost", "avg_medication_cost"]}
    title="Average cost by risk tier"
    yAxisTitle="Cost (USD)"
    type="stacked"
/>