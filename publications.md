---
layout: default
title: Publications
permalink: /publications/
---

# Publications & Preprints

<div class="subtitle-text">Complete bibliography automatically fetched and updated in real time</div>

<div class="hero-banner">
  🎓 For citation metrics and co-author graphs, visit my <strong><a href="https://scholar.google.com/citations?user=W4sIgrIAAAAJ&hl=en" target="_blank">Google Scholar Profile</a></strong>.
</div>

<div id="pub-loading-state" class="callout">
  ⏳ Fetching complete publication history from Semantic Scholar...
</div>

<div id="publication-container" style="display: none;">
  <div style="margin-bottom: 1.5rem; display: flex; justify-content: space-between; align-items: center; flex-wrap: wrap; gap: 1rem;">
    <div id="pub-count-badge" class="badge-pill" style="font-size: 1rem; padding: 0.5rem 1rem;">
      Loading total papers...
    </div>
    <div>
      <input type="text" id="pub-search-input" placeholder="🔍 Filter publications..." style="padding: 0.5rem 1rem; border: 1px solid #cbd5e1; border-radius: 8px; font-size: 0.95rem; width: 260px;">
    </div>
  </div>

  <div id="publication-list"></div>
</div>

<script>
async function loadAllPublications() {
  const authorId = "47335744"; // Julian Wolfson Semantic Scholar ID
  const apiUrl = `https://api.semanticscholar.org/graph/v1/author/${authorId}/papers?fields=title,authors,year,venue,externalIds,citationCount,publicationDate&limit=500`;

  try {
    const response = await fetch(apiUrl);
    if (!response.ok) throw new Error("Failed to load publication data");
    
    const data = await response.json();
    let papers = data.data || [];

    // Sort papers by year (descending) and title
    papers.sort((a, b) => {
      const yearA = a.year || 0;
      const yearB = b.year || 0;
      if (yearB !== yearA) return yearB - yearA;
      return a.title.localeCompare(b.title);
    });

    document.getElementById("pub-loading-state").style.display = "none";
    document.getElementById("publication-container").style.display = "block";
    document.getElementById("pub-count-badge").innerHTML = `📚 Total Indexed Publications: <strong>${papers.length}</strong>`;

    function renderPapers(filteredPapers) {
      const listContainer = document.getElementById("publication-list");
      if (filteredPapers.length === 0) {
        listContainer.innerHTML = "<p>No matching publications found.</p>";
        return;
      }

      let currentYear = null;
      let html = "";

      filteredPapers.forEach((paper) => {
        const paperYear = paper.year || "Preprints & Other";
        
        // Add year section header
        if (paperYear !== currentYear) {
          currentYear = paperYear;
          html += `<h2 style="margin-top: 2rem; margin-bottom: 1rem; color: #7a0019; border-bottom: 2px solid #e2e8f0; padding-bottom: 0.3rem;">${currentYear}</h2>`;
        }

        // Format authors (bold 'Julian Wolfson' or 'J Wolfson')
        const authorList = paper.authors.map(a => {
          if (a.name.includes("Wolfson")) {
            return `<strong>${a.name}</strong>`;
          }
          return a.name;
        }).join(", ");

        const venue = paper.venue ? `<em>${paper.venue}</em>` : "<em>Journal Article / Preprint</em>";
        const citations = paper.citationCount ? ` &bull; <span style="color: #64748b; font-size: 0.85rem;">Citations: ${paper.citationCount}</span>` : "";
        
        // External link (DOI or Semantic Scholar page)
        let linkHtml = "";
        if (paper.externalIds && paper.externalIds.DOI) {
          linkHtml = ` &bull; <a href="https://doi.org/${paper.externalIds.DOI}" target="_blank" style="font-size: 0.85rem;">DOI ↗</a>`;
        } else if (paper.externalIds && paper.externalIds.ArXiv) {
          linkHtml = ` &bull; <a href="https://arxiv.org/abs/${paper.externalIds.ArXiv}" target="_blank" style="font-size: 0.85rem;">arXiv ↗</a>`;
        }

        html += `
          <div class="card" style="margin-bottom: 1rem; padding: 1.1rem 1.3rem;">
            <div style="font-weight: 700; font-size: 1.05rem; color: #1e293b; margin-bottom: 0.35rem;">
              ${paper.title}
            </div>
            <div style="font-size: 0.95rem; color: #334155; margin-bottom: 0.35rem;">
              ${authorList}
            </div>
            <div style="font-size: 0.9rem; color: #64748b;">
              ${venue} (${paper.year || 'n.d.'})${citations}${linkHtml}
            </div>
          </div>
        `;
      });

      listContainer.innerHTML = html;
    }

    // Initial render
    renderPapers(papers);

    // Search filter listener
    document.getElementById("pub-search-input").addEventListener("input", (e) => {
      const query = e.target.value.toLowerCase();
      const filtered = papers.filter(p => 
        p.title.toLowerCase().includes(query) || 
        p.authors.some(a => a.name.toLowerCase().includes(query)) ||
        (p.venue && p.venue.toLowerCase().includes(query)) ||
        (p.year && p.year.toString().includes(query))
      );
      renderPapers(filtered);
    });

  } catch (err) {
    console.error(err);
    document.getElementById("pub-loading-state").className = "callout";
    document.getElementById("pub-loading-state").innerHTML = "⚠️ Unable to load live publications automatically. Please visit <a href='https://scholar.google.com/citations?user=W4sIgrIAAAAJ&hl=en' target='_blank'>Google Scholar</a> for the complete list.";
  }
}

loadAllPublications();
</script>