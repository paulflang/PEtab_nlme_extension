# PEtab NMLE extension

> [!NOTE]  
> An NLME extension will be most useful in conjunction with a [dosing](https://github.com/PEtab-dev/PEtab/issues/564) or [time-course](https://github.com/dilpath/petab_timecourse) extension.

This is a draft for a PEtab NLME extension. It adds the following to the existing PEtab specification.

## Individual table

This table is an addition to the existing PEtab format. It introduces a classification of individuals (most fine grained grouping) into several (more coarse grained) groups. Random effects can be specific to each individual or to the groups they belong to. Additionally, the individual table allows to speficy covariates. While covariates covariates could technically also be specified in the existing condition table, pharmacologists are used to specifying them along with the data pertaining to individuals. This specification also facilitates the use of covariates in more complex mathematical expressions (Elba, let's discuss).

### Detailed field description

- `individualId` [STRING]
- `${groupId}` [STRING, OPTIONAL]

   Further columns may be used to group individuals into different categories.


## Parameter table

In addition to the existing descriptions of fixed effects, the NLME extension adds the following columns to describe random effects.

### Detailed field description

- `randomVariableId` [STRING]

   ID of the random variable. If left empty, there is no random effect defined on the parameter of this row.

- `randomEffectFormula` [STRING]

   Random effect function as plain text formula expression. Must contain the `parameterId` and the `randomVariableId` and must not contain any other symbols (or should we allow that???).

-  `distributionType` [STRING: 'normal' or 'laplace', OPTIONAL]

   Assumed random effect distribution. Only normal and laplace distributions are currently allowed (but we could allow any term in some ontology for distributions). Defaults to normal. Distributions are assumed to be zero-centered. Covariances are defined in the covariance table.

- `groupType` [STRING]

   Defines how random effects are grouped together. Any column name from the individual table is allowed, except for covariate (i.e. numeric) columns.

## Covariance tables

These tables are an addition to the existing PEtab format to describe the covariances of the random effects. Each `groupType` for which covariances are estimated has its own table. Four formats are allowed

- Matrix format

  All random effects occur in columns and rows. Covariances that are estimate contain the value `1`, covariances that are assumed to be zero contain the value `0`, or are left blank.

- Sparse matrix format

  This format contains two columns, `randomEffect1` and `randomEffect2` to define sets (i.e. pairs) of random effects for which covariances shall be estimated. Variances are estimated for all random effects. No duplicate sets are allowed (e.g. if one row specifies `u1, u2`, there must not be a `u1, u1` row).

- Single cell format

  If the covariance table only contains a single cell, this cell must contain the string `full` to specify that all (co)variances are estimated.

- No table

  If no table is provided, only the variances, but no covariances are estimated.
  
## Measurement table

In addition to the existing descriptions of fixed effects, the NLME extension adds the following columns:

- `$covariateId` one column per distinct covariate
  
## Covariate table

This table describes the addition of covariate table in the PEtab NLME extension.

### Detailed field description

- `covariateId` [STRING]

   ID of the covariate.

- `covariateParameter` [STRING]

   Defines the parameter where the covariate effect (COVEFF) is modelled. It must be a `parameterId` defined in the parameter table. The COVEFF would be multiplied with the `parameterId`.

-  `covariateFormula` [STRING: 'exclude' or 'fractional' or 'power' or free text formula]

   Covariate effect function as plain text formula expression. The user can define its own formula or can use predefined models: 'exclude' or 'fractional' or 'power'.
   - 'exclude' --> Covariate not included on the parameter.
   $\text{COVEFF}=1$

   - 'fractional' --> Linear function. For categorical and continuous covariates.
   
   For continuous covariates:
   $\text{COVEFF}= 1 + \theta_{cov} \times \left(\text{covariateId} - median(\text{covariateId})\right)$
   
   For categorical covariates:
   
   $\text{if }  covariateValue  ==  Most common; \quad  \text{COVEFF} = 1$
   
   $\text{else }  ; \quad \text{COVEFF} = ( 1 + \theta_{cov})$

   - 'power' --> Power function. Continuous covariates only.
   $\text{COVEFF}= {\left(\frac{\text{covariateId}}{median(\text{covariateId})}\right)}^{\theta_{cov}}$
   
   Note that free text formula is also allowed

- `covariateType` [STRING]

   Defines whether a covariate is categorical or numerical.

- `covariateScale` [STRING: 'lin' or 'log' or 'log10']

   Same use as in the parameter table.

- `lowerBound` [NUMERICAL]

   Same use as in the parameter table.
   
- `upperBound` [NUMERICAL]

   Same use as in the parameter table.
   
- `nominalValue` [NUMERICAL]

   Same use as in the parameter table.
   
- `estimate` [NUMERICAL: 0 or 1]

   Same use as in the parameter table.
