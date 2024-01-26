# PEtab NMLE extension

> [!NOTE]  
> An NLME extension will be most useful in conjunction with a [dosing](https://github.com/PEtab-dev/PEtab/issues/564) or [time-course](https://github.com/dilpath/petab_timecourse) extension.

This is a draft for a PEtab NLME extension. It adds the following to the existing PEtab specification.

## Distribution table

This table is an addition to the existing PEtab format. It can be used to introduce a classification of e.g. individuals (most fine grained grouping) into several (more coarse grained) groups. Random effects can be specific to each individual or to the groups they belong to.

### Detailed field description

- `${distributionId}` [STRING, OPTIONAL]

   Each column lists the possible values for the given distribution.

## Random effect table

In addition to the existing descriptions of fixed effects in the parameter table, the NLME extension adds a random effect table to describe random effects.

### Detailed field description

- `randomVariableId` [STRING]

   ID of the random variable.

- `estimate` [0|1]

   1 or 0, depending on, if the parameter is estimated (1) or set to zero (to shift a random effect away from zero, see `randomEffectFormula`).

- `parameterId` [STRING]

   ID of the associated fixed effect.

- `randomEffectFormula` [STRING]

   Random effect function as plain text formula expression. Must contain the `parameterId` and the `randomVariableId`. May contain other `randomVariableId`s from other distributions. Must not contain any other symbols (or should we allow that???).

-  `distributionType` [STRING: 'normal' or 'laplace', OPTIONAL]

   Assumed random effect distribution. Only normal and laplace distributions are currently allowed (but we could allow any term in some ontology for distributions). Defaults to normal. Distributions are assumed to be zero-centered. Covariances are defined in the covariance table.

- `groupType` [STRING]

   Defines how random effects are grouped together. Any column name from the individual table is allowed, except for covariate (i.e. numeric) columns.

## Covariance tables

These tables are an addition to the existing PEtab format to describe the covariances of the random effects. Each `groupType` for which covariances are estimated has its own table. Four formats are allowed

- Matrix format

  All random effects occur in columns and rows. Covariances that are estimate contain the value `1`, covariances that are assumed to be zero contain the value `0`, or are left blank.

- Sparse matrix format

  This format contains two columns, `randomEffect1` and `randomEffect2` to list pairs of random effects for which covariances shall be estimated. Variances are estimated for all random effects. No duplicate sets are allowed (e.g. if one row specifies `u1, u2`, there must not be a `u2, u1` row).

- Single cell format

  If the covariance table only contains a single cell, this cell must contain the string `full` to specify that all (co)variances are estimated.

- No table

  If no table is provided, only the variances, but no covariances are estimated.
