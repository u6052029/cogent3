.. jupyter-execute::
    :hide-code:

    import set_working_directory

Likelihood analysis of multiple loci
====================================

.. sectionauthor:: Gavin Huttley

We want to know whether an exchangeability parameter is different between alignments. We will specify a null model, under which each alignment get's it's own motif probabilities and all alignments share branch lengths and the exchangeability parameter kappa (the transition / transversion ratio). We'll split the example alignment into two-pieces.

.. jupyter-execute::
    :linenos:

    from cogent3 import load_aligned_seqs, make_tree, make_table
    from cogent3.evolve.models import HKY85
    from cogent3.recalculation.scope import EACH, ALL
    from cogent3.maths.stats import chisqprob

    aln = load_aligned_seqs("data/long_testseqs.fasta")
    half = len(aln) // 2
    aln1 = aln[:half]
    aln2 = aln[half:]

We provide names for those alignments, then construct the tree, model instances.

.. jupyter-execute::
    :linenos:

    loci_names = ["1st-half", "2nd-half"]
    loci = [aln1, aln2]
    tree = make_tree(tip_names=aln.names)
    mod = HKY85()

To make a likelihood function with multiple alignments we provide the list of loci names. We can then specify a parameter (other than length) to be the same across the loci (using the imported ``ALL``) or different for each locus (using ``EACH``). We conduct a LR test as before.

.. jupyter-execute::
    :linenos:

    lf = mod.make_likelihood_function(tree, loci=loci_names, digits=2, space=3)
    lf.set_param_rule("length", is_independent=False)
    lf.set_param_rule("kappa", loci=ALL)
    lf.set_alignment(loci)
    lf.optimise(show_progress=False)
    lf

.. jupyter-execute::
    :linenos:

    all_lnL = lf.lnL
    all_nfp = lf.nfp
    lf.set_param_rule("kappa", loci=EACH)
    lf.optimise(show_progress=False)
    lf

.. jupyter-execute::
    :linenos:

    each_lnL = lf.lnL
    each_nfp = lf.nfp
    LR = 2 * (each_lnL - all_lnL)
    df = each_nfp - all_nfp

Just to pretty up the result display, I'll print(a table consisting of the test statistics created on the fly.)

.. jupyter-execute::
    :linenos:

    make_table(
        header=["LR", "df", "p"], rows=[[LR, df, chisqprob(LR, df)]], digits=2, space=3,
    )