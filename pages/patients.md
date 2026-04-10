# Patient drilldown

```sql patients
select
    patient_id,
    age_years,
    gender,
    RACE,
    city,
    unique_conditions,
    active_conditions,
    active_medications,
    total_encounters,
    inpatients_encounters,
    emergency_encounters,
    total_readmissions,
    readmission_rate_pct,
    round(total_encounter_cost, 0) as total_encounter_cost,
    round(total_medication_cost, 0) as total_medication_cost,
    case
        when readmission_rate_pct = 0 then 'Low'
        when readmission_rate_pct <= 50 then 'Medium'
        else 'High'
    end as risk_tier,
    is_high_risk,
    is_deceased
from mart_patient_summary
order by total_encounter_cost desc
```

## Patient population

<BigValue
    data={patients}
    value="patient_id"
    title="Total patients"
/>

<DataTable
    data={patients}
    search=true
    rows=15
/>

---

## Age distribution

```sql age_distribution
select
    case
        when age_years < 18 then 'Under 18'
        when age_years < 35 then '18-34'
        when age_years < 50 then '35-49'
        when age_years < 65 then '50-64'
        when age_years < 80 then '65-79'
        else '80+'
    end as age_group,
    count(*) as patients,
    sum(case when is_high_risk then 1 else 0 end) as high_risk_patients,
    round(avg(total_encounter_cost), 0) as avg_cost
from mart_patient_summary
group by age_group
order by min(age_years)
```

<BarChart
    data={age_distribution}
    x="age_group"
    y="patients"
    title="Patients by age group"
    yAxisTitle="Patients"
/>

---

## Gender breakdown

```sql gender_breakdown
select
    gender,
    count(*) as patients,
    round(avg(age_years), 1) as avg_age,
    round(avg(total_encounter_cost), 0) as avg_cost,
    sum(case when is_high_risk then 1 else 0 end) as high_risk_patients
from mart_patient_summary
group by gender
```

<DataTable data={gender_breakdown} />

---

## Race breakdown

``` sql race_breakdown
select
    RACE,
    count(*) as patients,
    round(avg(age_years), 1) as avg_age,
    round(avg(total_encounter_cost), 0) as avg_cost,
    sum(case when is_high_risk then 1 else 0 end) as high_risk_patients
from mart_patient_summary
group by race
order by patients desc
```

<BarChart 
    data={race_breakdown}
    x="RACE"
    y="patients"
    title="Patients by race"
    yAxisTitle="Patients"
/>