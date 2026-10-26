Written by Claude. I haven't checked the code, so use at your own peril.

<h1> Claude's explanation of what it does: How the Genotype-Calling App Simulates GTseq Data </h1>

<h2 id="overview">Overview</h2>
<p>The app really simulates every fish's genotypes and read counts. The
one big shortcut is that reads are never simulated as DNA sequences: for
each fish at each locus, the app draws how many reads carry allele A and
how many carry allele B. Everything after that (calling, filtering,
statistics) runs on those counts, just as the GTseq genotyper does on
counts from real reads.</p>
<p>With the defaults, that is 40 fish × 80 loci × 2 replicates = 6,400
genotypes, standing in for about 380,000 reads.</p>
<h2 id="simulating-the-fish">Simulating the fish</h2>
<h3 id="1-allele-frequencies-at-each-locus">1. Allele frequencies at
each locus</h3>
<ul>
<li><strong>Ancestral frequency.</strong> Each locus first gets a
starting allele frequency. By default this comes from a U-shaped beta
distribution, so many loci have a rare allele, kept between 0.02 and
0.98. The alternative setting is common alleles only, between 0.1 and
0.9.</li>
<li><strong>Population frequencies.</strong> Each population's frequency
is drawn independently around the ancestral one from a beta distribution
whose spread is set by the true FST. This is the Balding–Nichols model,
a standard way to simulate drift between populations from a common
ancestor.</li>
<li><strong>Monomorphic loci.</strong> A set share of loci (15% by
default) is fixed for one allele in both populations, so any
minor-allele reads there can only come from errors.</li>
</ul>
<h3 id="2-true-genotypes">2. True genotypes</h3>
<p>Each fish's genotype at each locus is drawn as the number of A
alleles (0, 1 or 2) from a binomial with its population's allele
frequency. Each population is therefore in Hardy–Weinberg equilibrium,
and loci are independent: there is no linkage. That is simpler than the
herring panel, where loci cluster on a few chromosomes.</p>
<h2 id="simulating-the-reads">Simulating the reads</h2>
<h3 id="3-how-many-reads-each-fish-gets-at-each-locus">3. How many reads
each fish gets at each locus</h3>
<ul>
<li><strong>Fish effect.</strong> Each fish gets a multiplier for DNA
quality, drawn from a log-normal distribution with mean 1. The "fold
spread" slider sets its standard deviation on the log scale (log 3 by
default), so some fish get far fewer reads.</li>
<li><strong>Locus effect.</strong> Each locus gets an amplification
multiplier the same way (fold spread 2.75 by default).</li>
<li><strong>Expected reads</strong> = average reads × fish multiplier ×
locus multiplier.</li>
<li><strong>Actual reads</strong> are drawn separately for each
replicate from a negative binomial around that expected value. That adds
the extra randomness of PCR on top of sampling, so read counts vary more
than pure chance would give.</li>
</ul>
<p>The two replicates of a fish share its DNA quality and its true
genotype, but get separate draws for read depth, allele balance and
errors, like two independent libraries made from the same extract.</p>
<h3 id="4-allele-balance-in-heterozygotes">4. Allele balance in
heterozygotes</h3>
<p>In a homozygote, every template molecule carries the same allele. In
a heterozygote, the share of template carrying A is drawn from a beta
distribution centered on 0.5. Its spread is the allele-balance noise
setting (SD 0.08 by default), standing in for unequal PCR amplification
of the two alleles.</p>
<h3 id="5-sequencing-error-and-the-read-counts">5. Sequencing error and
the read counts</h3>
<p>Each read shows the wrong allele with probability <em>e</em> (the
error slider). The chance that a read shows A is therefore (A share × (1
− <em>e</em>)) + (B share × <em>e</em>). The number of A reads is drawn
from a binomial with that probability and the read depth; the rest are B
reads.</p>
<p>This binomial draw is the main shortcut, and it costs nothing:
drawing each read separately with the same probability and adding them
up gives exactly the same distribution.</p>
<h2 id="from-reads-to-results">From reads to results</h2>
<h3 id="6-calling-genotypes">6. Calling genotypes</h3>
<p>The A and B counts go through the same rules as the GTseq
genotyper:</p>
<table>
<thead>
<tr class="header">
<th>Read counts</th>
<th>Call</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Fewer reads than the minimum</td>
<td>No call</td>
</tr>
<tr class="even">
<td>Ratio of A to B at or above the homozygote cutoff</td>
<td>A/A</td>
</tr>
<tr class="odd">
<td>Ratio at or below 1 ÷ homozygote cutoff</td>
<td>B/B</td>
</tr>
<tr class="even">
<td>Ratio between 1 ÷ heterozygote cutoff and the heterozygote
cutoff</td>
<td>A/B</td>
</tr>
<tr class="odd">
<td>Anything else</td>
<td>No call</td>
</tr>
</tbody>
</table>
<p>The headline numbers are then simple counts. Missing data is the
share of uncalled genotypes. Replicate discordance is the share of
pairs, both called, whose replicates disagree. Error compares the called
genotypes with the true ones.</p>
<h3 id="7-filtering-two-populations-tab-replicate-1-only">7. Filtering
(Two populations tab, replicate 1 only)</h3>
<ol type="1">
<li>Loci are removed if their call rate is below the loci threshold, or
if the minor allele frequency of their called genotypes is below the MAF
threshold.</li>
<li>Fish are removed if their call rate on the remaining loci is below
the fish threshold.</li>
</ol>
<h3 id="8-statistics">8. Statistics</h3>
<ul>
<li><strong>FST:</strong> Hudson's estimator, summed over loci before
dividing ("ratio of averages"), using only called genotypes. Each locus
uses its own sample sizes, so missing data adds noise but no direct
bias.</li>
<li><strong>He, Ho and FIS:</strong> He is 2<em>pq</em> averaged over
loci, Ho is the share of called genotypes that are heterozygous, and FIS
= 1 − Ho/He. Each is computed within populations and then averaged over
the two.</li>
<li><strong>PCA:</strong> genotypes are standardized by locus, with
missing genotypes filled with the locus average; that is why poorly
genotyped fish drift to the middle. The called PCA's axes are flipped if
needed to line up with the true PCA, because PCA axis signs are
arbitrary.</li>
<li><strong>False SNPs:</strong> loci that are truly monomorphic but
have a called minor allele frequency above 0 after filtering.</li>
</ul>
<h2 id="sliders-and-simplifications">Sliders and simplifications</h2>
<h3 id="9-what-happens-when-you-move-a-slider">9. What happens when you
move a slider</h3>
<ul>
<li><strong>Calling and filtering sliders</strong> only recompute steps
6 to 8 on the same read counts, so you compare settings on identical
data.</li>
<li><strong>Sequencing and population settings</strong> re-simulate from
the same random seed. The fish and their genotypes stay the same, and
only what the slider controls changes.</li>
<li><strong>"Simulate a new data set"</strong> changes the seed, giving
new fish.</li>
</ul>
<h3 id="what-the-model-leaves-out">What the model leaves out</h3>
<ul>
<li><strong>Sequences:</strong> reads exist only as counts of A and B,
with no bases, quality scores, primers or probes.</li>
<li><strong>Error types:</strong> errors to a third base, PCR errors,
index hopping and contamination are all folded into one wrong-allele
rate.</li>
<li><strong>Population structure:</strong> loci are unlinked and each
population is in Hardy–Weinberg equilibrium.</li>
<li><strong>Problem loci and samples:</strong> there are no primer
dimers, null alleles, duplicated loci or contaminated fish.</li>
</ul>
