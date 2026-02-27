---
name: Home
assetId: 3e265429-96e7-4b8a-8b4e-3e3fb3fe94b7
type: page
---

```sql main
select distinct
    triage_id,
    triage_user_first_name || ' ' || triage_user_last_name as triage_user_full_name,
    toISOYear(triage_date) as iso_year,
    triage_medical_unit_sor_region,
    triage_medical_unit_id,
    triage_medical_unit_name,
    toDate(triage_date) as triage_date,
    lower(hex(MD5(toString(triage_user_id)))) as triage_user_id,
    triage_occupation as triage_user_affiliation_occupation,
    triage_speciality as triage_user_affiliation_speciality,
    patient_id,
    skin_condition_id,
    triage_diagnosis,
    triage_action,
    triage_draft,
    triage_difficulty,
    toDate(tele_assessment_date) as tele_assessment_date,
    tele_assessment_diagnosis,
    tele_assessment_proposed_clinical_management_action
from dermloop_triage_data
where
    skin_condition_type = 'Rash' and
    triage_federation_name = 'DK'
order by toDate(triage_date), triage_medical_unit_name
```

{% dropdown
  id="unitselect"
  data="main"
  value_column="triage_medical_unit_name"
  initial_value="Læge Michael J. Bregnbak"
/%}

# Pilot Øvrige Hudlidelser - Datarapport
Klinik: {{unitselect}}

{% table title="1. Hvor mange sager er kladde og hvor mange er færdigregistreret?"
  data="main"
  filters=["unitselect"]
  page_size=200
  total_label="Hovedtotal"
  subtotal_position="top"
  show_subtotal_columns=false
  wrap_titles=false
%}
  {% dimension
    value="case when triage_draft then 'Færdigregistreret' when not triage_draft then 'Kladde' end"
    title=""
  /%}
  {% dimension
    value="triage_user_id as 'Bruger'"
  /%}
  {% pivot
    value="iso_year"
    sort="desc"
  /%}
  {% pivot
    value="formatDateTime(triage_date, '%V')"
    sort="desc"
  /%}
  {% measure
    value="count(distinct triage_id)"
    fmt="num0"
  /%}
{% /table %}

{% table title="2. Hvilke diagnoser har I registreret?"
  data="main"
  filters=["unitselect"]
  page_size=200
  total_label="Hovedtotal"
%}
  {% dimension
    value="triage_diagnosis"
    sort="asc"
  /%}
  {% measure
    value="count(distinct triage_id)"
  /%}
{% /table %}

{% table title="3. Overblik over sager sendt til televurdering"
  data="main"
  filters=["unitselect"]
  page_size=200
  total_label="Hovedtotal"
  where="triage_action = 'Telehenvisning'"
  order="triage_date"
%}
  {% dimension
    value="formatDateTime(triage_date, '%d/%m/%Y')"
    title="Dato (afsendelse)"
  /%}
  {% dimension
    value="triage_diagnosis"
    title="Diagnose (Almen praksis)"
  /%}
  {% dimension
    value="triage_action"
    title="Klinisk handling (almen praksis)"
  /%}
  {% dimension
    value="formatDateTime(tele_assessment_date, '%d/%m/%Y')"
    title="Dato (televurdering)"
  /%}
  {% dimension
    value="tele_assessment_diagnosis"
    title="Teledermatologens diagnose"
  /%}
  {% dimension
    value="tele_assessment_proposed_clinical_management_action"
    title="Teledermatologens foreslåede handling"
  /%}
{% /table %}

