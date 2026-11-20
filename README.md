# Technical Writing & SEO Architecture Portfolio
**Author:** Eng. Shady Haseeb (Shady Muhammad Ali Hasib)  
**Focus:** Technical SEO, Generative Engine Optimization (GEO), WordPress Architecture & Search Analytics

---

## 1. WordPress Technical SEO & GEO: Automating Clean Schema & Crawl Headers

When optimizing WordPress sites for both traditional search engines and Generative Engine Optimization (GEO), clean structured data and predictable HTTP/HTML signals help crawlers and AI answer engines parse entities without bloating page weight.

* **Entity-First Schema Injection:** Rather than stacking heavy multi-purpose plugins that output redundant JSON-LD blocks, hook a lightweight function into `wp_head` only on singular posts/pages where the entity applies.
* **Crawl & Cache Alignment:** Keep HTML output deterministic so edge caching (such as Cloudflare) and page-cache plugins serve identical markup to both users and search bots.
* **Measurable Markup Overhead:** A minimal `Organization` + `Article` JSON-LD payload typically adds only **~450–700 bytes** uncompressed (roughly **~250–350 bytes** over gzip/Brotli), keeping transfer overhead negligible while clarifying authorship, dates, and topical entities.

```php
<?php
add_action('wp_head', function () {
    if (!is_singular('post')) {
        return;
    }

    $post_id = get_the_ID();
    $data = [
        '@context' => '[https://schema.org](https://schema.org)',
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
