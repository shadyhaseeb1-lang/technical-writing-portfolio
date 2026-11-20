//# technical-writing-portfolio
//Technical writing samples, WordPress SEO/GEO automation, and search analytics data by Eng. Shady Haseeb.

//Here are two technical writing samples tailored to your background in WordPress/SEO architecture and workflow automation, including the code and data behind the numbers in the second piece.

//1. WordPress Technical SEO & GEO: Automating Clean Schema & Crawl Headers
//When optimizing WordPress sites for both traditional search engines and Generative Engine Optimization (GEO), clean structured data and predictable HTTP/HTML signals help crawlers and AI answer engines parse entities //without bloating page weight.

//Entity-First Schema Injection: Rather than stacking heavy multi-purpose plugins that output redundant JSON-LD blocks, hook a lightweight function into wp_head only on singular posts/pages where the entity applies.

//Crawl & Cache Alignment: Keep HTML output deterministic so edge caching (such as Cloudflare) and page-cache plugins serve identical markup to both users and search bots.

//Measurable Markup Overhead: A minimal Organization + Article JSON-LD payload typically adds only ~450–700 bytes uncompressed (roughly ~250–350 bytes over gzip/Brotli), keeping transfer overhead negligible while //clarifying authorship, dates, and topical entities.

<?php
add_action('wp_head', function () {
    if (!is_singular('post')) {
        return;
    }

    $post_id = get_the_ID();
    $data = [
        '@context' => 'https://schema.org',
        '@type' => 'Article',
        'headline' => get_the_title($post_id),
        'datePublished' => get_the_date('c', $post_id),
        'dateModified' => get_the_modified_date('c', $post_id),
        'author' => [
            '@type' => 'Person',
            'name' => get_the_author_meta('display_name', get_post_field('post_author', $post_id)),
        ],
    ];

    echo '<script type="application/ld+json">' . wp_json_encode($data, JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES) . "</script>\n";
}, 20);


//2. Search Console & Content Velocity: Computing Period-over-Period Impression Lift
//When reporting SEO or content growth from Google Search Console exports (such as a Pages.csv export comparing a current window against a prior baseline window), computing exact impression deltas and share of total //impressions prevents cherry-picking single-day spikes.

//Baseline vs. Current Comparison: Match each URL across two equal-length windows (for example, 28 days prior vs. 28 days current) and calculate both absolute delta and percentage lift.

//Guard Against Zero-Baseline Noise: When a newly indexed URL moves from 0 impressions to 120 impressions, report the raw count (+120 impressions, new indexation) rather than an undefined percentage.

//Code Behind the Calculation (Python)

//import pandas as pd

# Expected columns in a GSC Pages comparison export:
# Page, Current Impressions, Previous Impressions, Current Clicks, Previous Clicks
df = pd.read_csv("gsc_pages_comparison.csv")

df["impression_delta"] = df["Current Impressions"] - df["Previous Impressions"]
df["impression_lift_pct"] = df.apply(
    lambda r: round(((r["Current Impressions"] - r["Previous Impressions"]) / r["Previous Impressions"]) * 100, 1)
    if r["Previous Impressions"] > 0
    else None,
    axis=1,
)

total_curr_impressions = df["Current Impressions"].sum()
df["share_of_total_pct"] = ((df["Current Impressions"] / total_curr_impressions) * 100).round(1)

top_movers = df.sort_values("impression_delta", ascending=False).head(3)
print(top_movers[["Page", "Previous Impressions", "Current Impressions", "impression_delta", "impression_lift_pct", "share_of_total_pct"]])

Sample Data & Output Behind the Numbers
Suppose a 28-day GSC comparison across 4 tracked URLs (total_curr_impressions = 10,000) produces:


Page	Prev. Impressions	Curr. Impressions	Delta	Lift %	Share of Total
/guides/local-seo-checklist/	1,250	3,550	+2,300	+184.0%	35.5%
/guides/geo-ai-search-basics/	2,000	3,100	+1,100	+55.0%	31.0%
/services/technical-seo-audit/	1,800	2,150	+350	+19.4%	21.5%
/blog/new-release-notes/	0	1,200	+1,200	N/A (New)	12.0%

From that dataset, /guides/local-seo-checklist/ accounts for 35.5% of total impressions (3,550 / 10,000) and a 184.0% lift ((3,550 - 1,250) / 1,250 * 100), while overall impressions across the four pages rose from 5,050 to 10,000 (+98.0%).

If you'd like to adapt either of these for a specific application question (such as fintech/payments documentation or Local SEO), let me know which role or prompt you're answering.




