Basic Recipe: Static Analysis of a Simple Reform
================================================
This is the recipe you should follow first.
Mastering this recipe is a prerequisite for all the other recipes in this cookbook.

**Ingredients**

[Policy reform](https://pslmodels.github.io/Tax-Calculator/reformA.json)

## Imports

.. code-block:: python3

    import taxcalc as tc
    import pandas as pd
    from bokeh.io import show, output_notebook


## Setup

Use publicly-available CPS input file.

NOTE: if you have access to the restricted-use IRS-SOI PUF-based input file
and you have that file (named 'puf.csv') located in the directory
where this script is located, then you can substitute the following
statement for the prior statement:

```
recs = Records()
```

    
.. code-block:: python3
		
    recs = tc.Records.cps_constructor()


Specify `Calculator` object for static analysis of current-law policy.

.. code-block:: python3
		
    pol = tc.Policy()
    calc1 = tc.Calculator(policy=pol, records=recs)

NOTE: `calc1` now contains a PRIVATE COPY of `pol` and a PRIVATE COPY of `recs`,
so we can continue to use `pol` and `recs` in this script without any
concern about side effects from `Calculator` method calls on `calc1`.

.. code-block:: python3
		
   CYR = 2020

Calculate aggregate current-law income tax liabilities for `CYR`.
   
.. code-block:: python3
		
    calc1.advance_to_year(CYR)
    calc1.calc_all()
    itax_rev1 = calc1.weighted_total('iitax')

Read JSON reform file and use (the default) static analysis assumptions.

.. code-block:: python3
		
    reform_filename = ('https://raw.githubusercontent.com/PSLmodels/Tax-Calculator/' +
		       'master/docs/cookbook/reformA.json')
    params = tc.Calculator.read_json_param_objects(reform_filename, None)

Specify Calculator object for static analysis of reform policy.

.. code-block:: python3
		
    pol.implement_reform(params['policy'])
    calc2 = tc.Calculator(policy=pol, records=recs)

## Calculate

Calculate reform income tax liabilities for `CYR`.

.. code-block:: python3
		
    calc2.advance_to_year(CYR)
    calc2.calc_all()
    itax_rev2 = calc2.weighted_total('iitax')

## Results

Print total revenue estimates for 2018.

*Estimates in billons of dollars rounded to nearest hundredth of a billion.*

.. code-block:: python3
		
    print('{}_CLP_itax_rev($B)= {:.3f}'.format(CYR, itax_rev1 * 1e-9))
    print('{}_REF_itax_rev($B)= {:.3f}'.format(CYR, itax_rev2 * 1e-9))


Generate several other standard results tables.

.. code-block:: python3
		
    # Aggregate diagnostic tables for CYR.
    clp_diagnostic_table = calc1.diagnostic_table(1)
    ref_diagnostic_table = calc2.diagnostic_table(1)

    # Income-tax distribution for CYR with CLP and REF results side-by-side.
    dist_table1, dist_table2 = calc1.distribution_tables(calc2, 'weighted_deciles')
    assert isinstance(dist_table1, pd.DataFrame)
    assert isinstance(dist_table2, pd.DataFrame)
    dist_extract = pd.DataFrame()
    dist_extract['funits(#m)'] = dist_table1['count']
    dist_extract['itax1($b)'] = dist_table1['iitax']
    dist_extract['itax2($b)'] = dist_table2['iitax']
    dist_extract['aftertax_inc1($b)'] = dist_table1['aftertax_income']
    dist_extract['aftertax_inc2($b)'] = dist_table2['aftertax_income']

    # Income-tax difference table by expanded-income decile for CYR.
    diff_table = calc1.difference_table(calc2, 'weighted_deciles', 'iitax')
    assert isinstance(diff_table, pd.DataFrame)
    diff_extract = pd.DataFrame()
    dif_colnames = ['count', 'tot_change', 'mean', 'pc_aftertaxinc']
    ext_colnames = ['funits(#m)', 'agg_diff($b)', 'mean_diff($)', 'aftertaxinc_diff(%)']
    for dname, ename in zip(dif_colnames, ext_colnames):
	diff_extract[ename] = diff_table[dname]

## Plotting

Generate a decile graph and display it using Bokeh (will render in Jupyter, not in webpage).

.. code-block:: python3
		
    fig = calc1.pch_graph(calc2)
    output_notebook()
    show(fig)

## Print tables

CLP diagnostic table for `CYR`.

.. code-block:: python3
		
    clp_diagnostic_table

REF diagnostic table for CYR.

.. code-block:: python3
		
    ref_diagnostic_table

Extract of CYR distribution tables by baseline expanded-income decile.

.. code-block:: python3
		
    dist_extract

Extract of CYR income-tax difference table by expanded-income decile.

.. code-block:: python3
		
    diff_extract