NAME: Evaluation of N-Gram Language models using SRILM Toolkit

ngram-count - count N-grams and estimate language models

SYNOPSIS

ngram-count [ -help ] option ...

DESCRIPTION

ngram-count generates and manipulates N-gram counts, and estimates N-gram language models from them. The program first builds an internal N-gram count set, either by reading counts from a file, or by scanning text input. Following that, the resulting counts can be output back to a file or used for building an N-gram language model in ARPA ngram-format(5). Each of these actions is triggered by corresponding options, as described below.

OPTIONS

Each filename argument can be an ASCII file, or a compressed file (name ending in .Z or .gz), or '''' to indicate stdin/stdout.

-help
Print option summary.

-version
Print version information.

-order n
Set the maximal order (length) of N-grams to count. This also determines the order of the estimated LM, if any. The default order is 3.

-vocab file
Read a vocabulary from file. Subsequently, out-of-vocabulary words in both counts or text are replaced with the unknown-word token. If this option is not specified all words found are implicitly added to the vocabulary.

-vocab-aliases file
Reads vocabulary alias definitions from file, consisting of lines of the form
	alias word
This causes all tokens alias to be mapped to word.

-write-vocab file
Write the vocabulary built in the counting process to file.

-write-vocab-index file
Write the internal word-to-integer mapping to file.

-tagged
Interpret text and N-grams as consisting of word/tag pairs.

-tolower
Map all vocabulary to lowercase.

-memuse
Print memory usage statistics.
Counting Options

-text textfile
Generate N-gram counts from text file. textfile should contain one sentence unit per line. Begin/end sentence tokens are added if not already present. Empty lines are ignored.

-text-has-weights
Treat the first field in each text input line as a weight factor by which the N-gram counts for that line are to be multiplied.

-text-has-weights-last
Similar to -text-has-weights but with weights at the ends of lines.

-no-sos
Disable the automatic insertion of start-of-sentence tokens in N-gram counting.

-no-eos
Disable the automatic insertion of end-of-sentence tokens in N-gram counting.

-read countsfile
Read N-gram counts from a file. Ascii count files contain one N-gram of words per line, followed by an integer count, all separated by whitespace. Repeated counts for the same N-gram are added. Thus several count files can be merged by using cat(1) and feeding the result to ngram-count -read - (but see ngram-merge(1) for merging counts that exceed available memory). Counts collected by -text and -read are additive as well. Binary count files (see below) are also recognized.

-read-google dir
Read N-grams counts from an indexed directory structure rooted in dir, in a format developed by Google to store very large N-gram collections. The corresponding directory structure can be created using the script make-google-ngrams described in training-scripts(1).

-intersect file
Limit reading of counts via -read and -read-google to a subset of N-grams defined by another count file. (The counts in file are ignored, only the N-grams are relevant.)

-write file
Write total counts to file.

-write-binary file
Write total counts to file in binary format. Binary count files cannot be compressed and are typically larger than compressed ascii count files. However, they can be loaded faster, especially when the -limit-vocab option is used.

-write-order n
Order of counts to write. The default is 0, which stands for N-grams of all lengths.

-writen file
where n is 1, 2, 3, 4, 5, 6, 7, 8, or 9. Writes only counts of the indicated order to file. This is convenient to generate counts of different orders separately in a single pass.

-sort
Output counts in lexicographic order, as required for ngram-merge(1).

-recompute
Regenerate lower-order counts by summing the highest-order counts for each N-gram prefix.

-limit-vocab
Discard N-gram counts on reading that do not pertain to the words specified in the vocabulary. The default is that words used in the count files are automatically added to the vocabulary.


LM Options

-lm lmfile
Estimate a language model from the total counts and write it to lmfile. This option applies to several LM model types (see below) but the default is to estimate a backoff N-gram model, and write it in ngram-format(5).

-write-binary-lm
Output lmfile in binary format. If an LM class does not provide a binary format the default (text) format will be output instead.

-nonevents file
Read a list of words from file that are to be considered non-events, i.e., that can only occur in the context of an N-gram. Such words are given zero probability mass in model estimation.

-float-counts
Enable manipulation of fractional counts. Only certain discounting methods support non-integer counts.

-skip
Estimate a skip N-gram model, which predicts a word by an interpolation of the immediate context and the context one word prior. This also triggers N-gram counts to be generated that are one word longer than the indicated order. The following four options control the EM estimation algorithm used for skip-N-grams.


-init-lm lmfile
Load an LM to initialize the parameters of the skip-N-gram.

-skip-init value
The initial skip probability for all words.

-em-iters n
The maximum number of EM iterations.

-em-delta d
The convergence criterion for EM: if the relative change in log likelihood falls below the given value, iteration stops.

-count-lm
Estimate a count-based interpolated LM using Jelinek-Mercer smoothing (Chen & Goodman, 1998). Several of the options for skip-N-gram LMs (above) apply. An initial count-LM in the format described in ngram(1) needs to be specified using -init-lm. The options -em-iters and -em-delta control termination of the EM algorithm. Note that the N-gram counts used to estimate the maximum-likelihood estimates come from the -init-lm model. The counts specified with -read or -text are used only to estimate the smoothing (interpolation weights).

-maxent
Estimate a maximum entropy N-gram model. The model is written to the file specifed by the -lm option.
If -init-lm file is also specified then use an existing maxent model from file as prior and adapt it using the given data.

-maxent-alpha A
Use the L1 regularization constant A for maxent estimation. The default value is 0.5.

-maxent-sigma2 S
Use the L2 regularization constant S for maxent estimation. The default value is 6 for estimation and 0.5 for adaptation.

-maxent-convert-to-arpa
Convert the maxent model to ngram-format(5) before writing it out, using the algorithm by Wu (2002).

-unk
Build an ''open vocabulary'' LM, i.e., one that contains the unknown-word token as a regular word. The default is to remove the unknown word.

-map-unk word
Map out-of-vocabulary words to word, rather than the default <unk> tag.

-trust-totals
Force the lower-order counts to be used as total counts in estimating N-gram probabilities. Usually these totals are recomputed from the higher-order counts.

-prune threshold
Prune N-gram probabilities if their removal causes (training set) perplexity of the model to increase by less than threshold relative.

-minprune n
Only prune N-grams of length at least n. The default (and minimum allowed value) is 2, i.e., only unigrams are excluded from pruning.

-debug level
Set debugging output from estimated LM at level. Level 0 means no debugging. Debugging messages are written to stderr.

-gtnmin count
where n is 1, 2, 3, 4, 5, 6, 7, 8, or 9. Set the minimal count of N-grams of order n that will be included in the LM. All N-grams with frequency lower than that will effectively be discounted to 0. If n is omitted the parameter for N-grams of order > 9 is set.
NOTE: This option affects not only the default Good-Turing discounting but the alternative discounting methods described below as well.

-gtnmax count
where n is 1, 2, 3, 4, 5, 6, 7, 8, or 9. Set the maximal count of N-grams of order n that are discounted under Good-Turing. All N-grams more frequent than that will receive maximum likelihood estimates. Discounting can be effectively disabled by setting this to 0. If n is omitted the parameter for N-grams of order > 9 is set.
In the following discounting parameter options, the order n may be omitted, in which case a default for all N-gram orders is set. The corresponding discounting method then becomes the default method for all orders, unless specifically overridden by an option with n. If no discounting method is specified, Good-Turing is used.

-gtn gtfile
where n is 1, 2, 3, 4, 5, 6, 7, 8, or 9. Save or retrieve Good-Turing parameters (cutoffs and discounting factors) in/from gtfile. This is useful as GT parameters should always be determined from unlimited vocabulary counts, whereas the eventual LM may use a limited vocabulary. The parameter files may also be hand-edited. If an -lm option is specified the GT parameters are read from gtfile, otherwise they are computed from the current counts and saved in gtfile.

-cdiscountn discount
where n is 1, 2, 3, 4, 5, 6, 7, 8, or 9. Use Ney's absolute discounting for N-grams of order n, using discount as the constant to subtract.

-wbdiscountn
where n is 1, 2, 3, 4, 5, 6, 7, 8, or 9. Use Witten-Bell discounting for N-grams of order n. (This is the estimator where the first occurrence of each word is taken to be a sample for the ''unseen'' event.)

-ndiscountn
where n is 1, 2, 3, 4, 5, 6, 7, 8, or 9. Use Ristad's natural discounting law for N-grams of order n.

-addsmoothn delta
where n is 1, 2, 3, 4, 5, 6, 7, 8, or 9. Smooth by adding delta to each N-gram count. This is usually a poor smoothing method and included mainly for instructional purposes.

-kndiscountn
where n is 1, 2, 3, 4, 5, 6, 7, 8, or 9. Use Chen and Goodman's modified Kneser-Ney discounting for N-grams of order n.

-kn-counts-modified
Indicates that input counts have already been modified for Kneser-Ney smoothing. If this option is not given, the KN discounting method modifies counts (except those of highest order) in order to estimate the backoff distributions. When using the -write and related options the output will reflect the modified counts.

-kn-modify-counts-at-end
Modify Kneser-Ney counts after estimating discounting constants, rather than before as is the default.

-knn knfile
where n is 1, 2, 3, 4, 5, 6, 7, 8, or 9. Save or retrieve Kneser-Ney parameters (cutoff and discounting constants) in/from knfile. This is useful as smoothing parameters should always be determined from unlimited vocabulary counts, whereas the eventual LM may use a limited vocabulary. The parameter files may also be hand-edited. If an -lm option is specified the KN parameters are read from knfile, otherwise they are computed from the current counts and saved in knfile.

-ukndiscountn
where n is 1, 2, 3, 4, 5, 6, 7, 8, or 9. Use the original (unmodified) Kneser-Ney discounting method for N-grams of order n.

-interpolaten
where n is 1, 2, 3, 4, 5, 6, 7, 8, or 9. Causes the discounted N-gram probability estimates at the specified order n to be interpolated with lower-order estimates. (The result of the interpolation is encoded as a standard backoff model and can be evaluated as such -- the interpolation happens at estimation time.) This sometimes yields better models with some smoothing methods (see Chen & Goodman, 1998). Only Witten-Bell, absolute discounting, and (original or modified) Kneser-Ney smoothing currently support interpolation.

-meta-tag string
Interpret words starting with string as count-of-count (meta-count) tags. For example, an N-gram
	a b string3	4
means that there were 4 trigrams starting with "a b" that occurred 3 times each. Meta-tags are only allowed in the last position of an N-gram.
Note: when using -tolower the meta-tag string must not contain any uppercase characters.

-read-with-mincounts
Save memory by eliminating N-grams with counts that fall below the thresholds set by -gtNmin options during -read operation (this assumes the input counts contain no duplicate N-grams). Also, if -meta-tag is defined, these low-count N-grams will be converted to count-of-count N-grams, so that smoothing methods that need this information still work correctly.
