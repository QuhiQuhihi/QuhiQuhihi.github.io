# Quantitative Boxer

Research articles on portfolio construction, financial models, and quantitative investing
by Daham Kim. The [Research index](_tabs/research.md) introduces seven collections and links
articles to detailed notes, executed notebooks, or accompanying result tables.

| Collection | Source project |
| --- | --- |
| Robust asset allocation | [Allocation ideas and validation](https://github.com/QuhiQuhihi/project_Asset_Allocation) |
| QuantLib for FICC | [Pricing, curves, risk, and desk examples](https://github.com/QuhiQuhihi/project_FICC_Quant) |
| Market regimes | [Causal forecasting and retrospective segmentation](https://github.com/QuhiQuhihi/regime_model) |
| SVD and PCA | [Portfolio risk and covariance experiments](https://github.com/QuhiQuhihi/SVD_Portfolio_Strategy) |
| Sector factor models | [State Street ETFs, factor exposures, and robustness](https://github.com/QuhiQuhihi/Famma-French-Factors-with-Sector-ETF) |
| Corporate-bond factors | [Issuer equity information and incremental bond-portfolio value](https://github.com/QuhiQuhihi/Factor-Strategy-for-Corporate-Bond-) |
| Adaptive RFQ pricing | [Monotone demand, forgetting, and exploration](_posts/ML/2026-09-22-Adaptive_RFQ_Pricing.md) |

The September 2026 revision updates 22 existing articles while retaining their URLs and
original publication dates, then adds corporate-bond and adaptive RFQ research articles.
Research figures are unchanged copies of verified project outputs.
[Publication provenance](tools/research-publication.json) records source commits where
available, document and figure hashes, notebook output locations, and result-table exports.
Historical inputs remain in the research projects' permitted local caches; they are not
bundled with this site.

The adaptive RFQ article uses anonymized prose, five original figures, and four aggregate
result tables. Course, instructor, and institution identifiers are omitted. The original
manuscript, notebook, workbook, and submission archive are not bundled with the article.

## Preview and check

This site uses Jekyll with the [Chirpy theme](https://github.com/cotes2020/jekyll-theme-chirpy).
Ruby 3.3 and `Gemfile.lock` define the checked build environment.

```sh
bundle install
JEKYLL_ENV=production bundle exec jekyll build
bundle exec htmlproofer _site --disable-external --check-html --allow_hash_href
bundle exec jekyll serve
```

The link check covers generated HTML and local targets. It does not verify live external
sources or rerun the research computations. Use each linked project's executed notebooks
and validation record for numerical evidence.

The Pages workflow builds and checks pushes to `main` before deployment, following
[GitHub's current workflow interface](https://docs.github.com/en/pages/getting-started-with-github-pages/using-custom-workflows-with-github-pages).
The math helper uses the documented [cdnjs polyfill mirror](https://blog.cloudflare.com/polyfill-io-now-available-on-cdnjs-reduce-your-supply-chain-risk/).
Files under `tools/` record repository provenance and are excluded from the generated site.

The starting blog commit is preserved on `old`. The original [license](LICENSE) and
Chirpy attribution are retained.
