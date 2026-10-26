Written by Claude. I haven't checked the code, so use at your own peril, and use for teaching only.

<h1 id="gtseq-read-explorer">GTseq Read Explorer</h1>
<p>An interactive web page that follows real GTseq reads from five
Pacific herring, from the raw FASTQ files to genotype calls. It was
built for an introductory genetics lab, so students can see what
sequencing data look like before working with genotypes.</p>

<h2 id="the-data">The data</h2>
<ul>
<li><strong>Fish:</strong> five Pacific herring (23OB-01, 23OB-02,
23OB-03, 23OB-04 and 23OB-23).</li>
<li><strong>Panel:</strong> the herring GTseq163 panel, with 163 SNP
loci.</li>
<li><strong>Sequencing:</strong> Illumina NextSeq, 76-base paired-end
reads, already sorted into one file per fish by the sequencing provider.
Fish 23OB-23 has read 1 only.</li>
<li><strong>Loci are named</strong> by their position in the herring
genome (chromosome:start-end).</li>
</ul>
<h2 id="what-the-page-shows">What the page shows</h2>
<h3 id="step-1-where-the-reads-go">Step 1: Where the reads go</h3>
<p>Each bar shows what happened to all reads from one fish:</p>
<table>
<thead>
<tr class="header">
<th>Category</th>
<th>Meaning</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>SNP amplicons</td>
<td>Reads from loci with at least one variable site in these five
fish</td>
</tr>
<tr class="even">
<td>Amplicons with no SNP</td>
<td>Reads from loci that amplified but did not vary in these fish</td>
</tr>
<tr class="odd">
<td>Primer dimers</td>
<td>Reads shorter than 50 bases that start with a primer: two primers
joined with no fish DNA between them</td>
</tr>
<tr class="even">
<td>Off-target or unreadable</td>
<td>Reads that were sorted to a locus but do not form a usable amplicon</td>
</tr>
<tr class="odd">
<td>No primer match</td>
<td>Reads that could not be assigned to any locus</td>
</tr>
</tbody>
</table>
<h3 id="step-2-one-locus-read-by-read">Step 2: One locus, read by
read</h3>
<p>Up to 16 randomly chosen reads per fish, sorted by the base at the
SNP. The forward primer is teal, the reverse primer violet, and the
panel SNP blue or orange. Bases that match the consensus can be shown as
dots, and bases that differ from it are red and underlined (mostly
sequencing errors). Extra variable sites that the panel does not target
are marked in gray on the amplicon map.</p>
<h3 id="step-3-probes-turn-reads-into-counts">Step 3: Probes turn reads
into counts</h3>
<p>The two 13-base probes for the panel SNP (a dot matches any base),
the number of reads that carry each allele in each fish, the ratio, the
genotype call, and a plot of allele 1 against allele 2 reads with the
calling cutoffs.</p>
<h3 id="step-4-genotypes-across-the-panel">Step 4: Genotypes across the
panel</h3>
<p>One column per locus and one row per fish, colored by genotype, with
each fish's observed heterozygosity. Clicking a cell opens that locus in
Step 2.</p>
<h2 id="how-the-data-were-processed">How the data were processed</h2>
<ol type="1">
<li><strong>Sort reads to loci.</strong> Reads were grouped by their
first 20 bases, allowing 2 mismatches. Groups with at least 100 reads
were treated as loci (180 groups).</li>
<li><strong>Trim and merge.</strong> Illumina adapter sequence was
trimmed. Read 1 and read 2 were merged into one sequence where they
overlap, giving the full amplicon.</li>
<li><strong>Find primer dimers.</strong> Reads shorter than 50 bases
that start with a primer were counted as primer dimers.</li>
<li><strong>Find SNPs.</strong> For each locus, a sequence was counted
as a real haplotype if it made up at least 20% of a fish's full-length
reads. Positions where haplotypes differ are variable sites.</li>
<li><strong>Match to the panel.</strong> Read groups were matched to the
official panel loci by their forward primer. Of the panel's 163 loci,
147 amplified well; 130 of these vary in the five fish and are shown on
the page, and 17 do not vary.</li>
<li><strong>Call genotypes.</strong> Following the GTseq genotyper, read
1 must start with the locus's forward primer, then reads containing
probe 1 or probe 2 are counted. Calls use the GTseq rules: at least 10
reads, a ratio of 10 or more for a homozygote, and 0.2 to 5 for a
heterozygote.</li>
</ol>
<h2 id="limitations">Limitations</h2>
<ul>
<li>The primer is shown as the first 20 bases of each read; the real
primer may be a few bases longer.</li>
<li>At 5 loci the official probe does not match the amplicon reads, so a
probe inferred from the reads is shown instead.</li>
<li>Fish 23OB-23 has read 1 only, so its reads stop at 76 bases and some
SNPs further along the amplicon cannot be called for it.</li>
<li>Only five fish are included, so SNPs that do not vary in these fish
appear as monomorphic.</li>
</ul>
<h2 id="files">Files</h2>
<ul>
<li><code>index.html</code>: the explorer. It is a single self-contained
page with the reads built in, so it needs no server or other files; it
loads its fonts from Google Fonts.</li>
</ul>
</body>
</html>

