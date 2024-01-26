# PEtab NMLE extension

> [!NOTE]  
> An NLME extension will be most useful in conjunction with a [dosing](https://github.com/PEtab-dev/PEtab/issues/564) or [time-course](https://github.com/dilpath/petab_timecourse) extension.

This is a draft for a PEtab NLME extension. It adds the following to the existing PEtab specification.

## YAML configuration file

In addition to existing attributes, this extension uses `covariance_files` to define distributions of random effects. Each distribution has a `distributionId`, `distributionType` (e.g. normal, laplace) and `covariance` (defined in a TSV file). Distributions have zero mean. The `randomEffectFormula` in the random effect table can be used to shift the location of the distribution.

## Random effect table

In addition to the existing descriptions of fixed effects in the parameter table, the NLME extension adds a random effect table to describe random effects.

### Detailed field description

- `randomVariableId` [STRING]

   ID of the random variable.

- `distributionId` [STRING]

   Specifies the Id of the distribution from which this random effect is drawn. Any Id from the distributions listed in the YAML configuration file is allowed.

- `estimate` [0|1]

   1 or 0, depending on, if the parameter is estimated (1) or set to zero (to shift a random effect away from zero, see `randomEffectFormula`).

- `parameterId` [STRING]

   ID of the associated fixed effect.

- `randomEffectFormula` [STRING]

   Random effect function as plain text formula expression. Must contain the `parameterId` and the `randomVariableId`. May contain other `randomVariableId`s from other distributions. Must not contain any other symbols (or should we allow that???).

## Covariance tables

These tables are an addition to the existing PEtab format to describe the covariances of the distribtions from which the random effects are drawn. Each `distributionId` has its own table. Four formats are allowed

- Matrix format

  All random effects occur in columns and rows. Covariances that are estimate contain the value `1`, covariances that are assumed to be zero contain the value `0`, or are left blank.

- Sparse matrix format

  This format contains two columns, `randomEffect1` and `randomEffect2` to list pairs of random effects for which covariances shall be estimated. Variances are estimated for all random effects. No duplicate sets are allowed (e.g. if one row specifies `u1, u2`, there must not be a `u2, u1` row).

- Single cell format

  If the covariance table only contains a single cell, this cell must contain the string `full` to specify that all (co)variances are estimated.

- No table

  If no table is provided, only the variances, but no covariances are estimated.

## Measurement table

In addition to existing collumns, the measurement table must contain every `distributionId` from the YAML configuration file as a column. Each data point (i.e. row) is assigned to one entity for each `distributionId`. This allows grouping of data points. As a simple example, the YAML configuration file defines just one `distributionId` that could be called `patient`. Rows that share the same entry in the `patient` column belong to the same patient. Therefore, they must share the same value for all random effects that are drawn from the `patient` distribution.
