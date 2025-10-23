# DAX Measures for Parcoursup Power BI Dashboard

This document presents all the **DAX measures** created for the **Power BI dashboard** based on the Parcoursup 2024 dataset. Each measure is grouped by analytical theme, with explanations of its purpose and logic.

---

## Project Context

The dashboard provides analytical insights on Parcoursup admissions data.

---

## Admission and Offers Measures

### **TotalAdmissions**

Counts all admitted students with a non-empty name field.

```DAX
TotalAdmissions = CALCULATE(
    COUNTROWS('Admissions'),
    NOT(ISBLANK('Admissions'[Nom Candidat]))
)
```

### **PropositionsAcceptéesJ3 / OuverturePC / Clôture / AprèsClôture**

Measures the number of offers accepted at different milestones of the admission timeline.

```DAX
PropositionsAcceptéesJ3 = CALCULATE(
    MAX('bdd-2024'[Dont propositions d'admission acceptées définitivement]),
    'bdd-2024'[Date extraction] <= DATE(2024, 6, 3)
)
```

(Same logic applied for other key dates like June 11, July 10, July 12, and September 13.)

### **PourcentagePropositionsAcceptées***

Computes the percentage of accepted offers over available seats.

```DAX
PourcentagePropositionsAcceptéesCloture =
VAR Pourcentage =
    CALCULATE(
        DIVIDE(
            MAX('bdd-2024'[Dont propositions d'admission acceptées définitivement]),
            MAX('bdd-2024'[Places]),
            0
        ) * 100,
        'bdd-2024'[Date extraction] <= DATE(2024, 9, 13)
    )
RETURN
    FORMAT(Pourcentage, "0") & "%"
```

---

## Ranking and Attractiveness

### **RangDernierAppelé***

Calculates the rank of the last candidate called for each admission stage.

```DAX
RangDernierAppeléCloture =
CALCULATE(
    MAX('bdd-2024'[Rang du dernier candidat appelé]),
    'bdd-2024'[Date extraction] <= DATE(2024, 9, 13)
)
```

### **PourcentageRangVoeuxJ3**

Shows the ratio of the last called rank to total applications.

```DAX
PourcentageRangVoeuxJ3 =
VAR Pourcentage =
    CALCULATE(
        DIVIDE(
            MAX('bdd-2024'[Rang du dernier candidat appelé]),
            MAX('Voeux'[Total des vœux Année N]),
            0
        ) * 100,
        'bdd-2024'[Date extraction] <= DATE(2024, 6, 3)
    )
RETURN
    FORMAT(Pourcentage, "0") & "%"
```

### **PourcentageAttractTexte**

Expresses the proportion of accepted offers relative to the total number of applications.

```DAX
PourcentageAttractTexte =
VAR Pourcentage = MAX('bdd-2024'[Dont propositions d'admission acceptées définitivement]) / SUM('Voeux'[Total des vœux Année N])
RETURN IF(ISBLANK(Pourcentage), "nd", FORMAT(Pourcentage, "0%"))
```

---

## Geographic Distribution

### **Académie Measures**

Count admissions by academic region.

```DAX
AcadémieParisTotal =
CALCULATE(
    COUNTROWS(Admissions),
    FILTER(Admissions, 'Admissions'[Département Etablissement origine - Libellé 2023/2024] = "Paris")
)
```

Other similar measures include:

* `AcadémieCréteilTotal`
* `AcadémieVersaillesTotal`
* `HorsIleDeFranceTotal`

---

## Socio-Professional Categories (CSP)

### **Category Counts**

Each CSP is identified by the first digit of the parent’s socio-professional code.

```DAX
CSP1CadresProfessionsIntellectuelles =
CALCULATE(
    COUNTROWS('Admissions'),
    LEFT('Admissions'[Responsable légal 1 - Code Catégorie Socio-professionnelle], 1) = "3"
)
```

(Repeat for categories: Agriculteurs, Artisans, Employés, Ouvriers, Professions Intermédiaires, Sans Activité, Non Renseigné.)

### **Percentage Distribution**

Calculates the percentage of each CSP group over all admissions.

```DAX
PourcentageCadresProfessionsIntellectuellesCSP1 =
DIVIDE(
    CALCULATE(
        COUNTROWS('Admissions'),
        LEFT('Admissions'[Responsable légal 1 - Code Catégorie Socio-professionnelle], 1) = "3"
    ),
    CALCULATE(COUNTROWS('Admissions'), ALL('Admissions')),
    0
) * 100
```

---

## Miscellaneous Calculations

### **FormationEnPC**

Indicates whether a program is part of the Parcoursup second phase "Phase complémentaire".

```DAX
FormationEnPC =
IF(
    CALCULATE(
        COUNTROWS(
            FILTER(
                'Formation-PC',
                'Formation-PC'[Code formation affectation] = MAX('rattachement_composantes'[Code formation affectation]) &&
                'Formation-PC'[enPC] = "OUI"
            )
        )
    ) > 0,
    "OUI",
    "NON"
)
```

### **RemplissageIntervalles**

Dynamic measure that computes the seat fill rate by time interval.

```DAX
RemplissageIntervalles =
SWITCH(
    TRUE(),
    MAX('bdd-2024'[Date extraction]) <= DATE(2024, 6, 3),
        DIVIDE(SUM('bdd-2024'[Dont propositions d'admission acceptées définitivement]), SUM('bdd-2024'[Places]), 0) * 100,
    MAX('bdd-2024'[Date extraction]) <= DATE(2024, 6, 11),
        DIVIDE(SUM('bdd-2024'[Dont propositions d'admission acceptées définitivement]), SUM('bdd-2024'[Places]), 0) * 100
)
```

---

## Notes

* All measures follow DAX best practices, using `CALCULATE`, `FILTER`, and variable scoping with `VAR`.
* Percentages are formatted as text for readability in Power BI cards.
* Date thresholds (June–September 2024) match Parcoursup admission stages.

---
