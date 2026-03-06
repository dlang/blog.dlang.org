---
layout: default
title: Search Results
permalink: /search/
hide_sidebar: true
---

<style>
  /* Fix for mobile cutoff and reflow */
  @media (max-width: 600px) {
    .search-title-row { flex-direction: column; align-items: flex-start !important; gap: 10px; margin-bottom: 20px; }
    .search-bar-container { flex-direction: column; }
    #search-button { width: 100%; padding: 15px !important; }
    .input-wrapper { width: 100%; }
  }
</style>

<div class="search-page-container" style="max-width: 800px; margin: 0 auto; padding: 20px;">
  
  <div class="search-title-row" style="display: flex; justify-content: space-between; align-items: baseline; flex-wrap: wrap; margin-bottom: 10px;">
    <h1 style="margin: 0;">Search Results</h1>
    <a id="return-link" href="#" style="display: none; color: #888; text-decoration: none; font-size: 0.9rem; white-space: nowrap;">&larr; Return to Article</a>
  </div>

  <div class="search-bar-container" style="display: flex; gap: 10px; margin-bottom: 20px;">
    <div class="input-wrapper" style="position: relative; flex-grow: 1;">
      <input type="text" id="search-input" placeholder="Search by title, tag, or year..." 
             style="width: 100%; box-sizing: border-box; padding: 12px; font-size: 1.1rem; border: 1px solid #ddd; border-radius: 4px; padding-right: 40px;">
      
      <button id="clear-search" style="position: absolute; right: 10px; top: 50%; transform: translateY(-50%); border: none; background: transparent; cursor: pointer; font-size: 1.5rem; color: #999; display: none;">&times;</button>
    </div>
    
    <button id="search-button" style="padding: 12px 24px; background: #b03931; color: white; border: none; border-radius: 4px; cursor: pointer; font-weight: bold; white-space: nowrap;">
      Search
    </button>
  </div>

  <div id="search-status" style="margin-bottom: 20px; font-style: italic; color: #666; min-height: 1.2em;"></div>
  
  <ul id="results-list" style="list-style: none; padding: 0;"></ul>
</div>

<script>
async function initSearch() {
  const input = document.getElementById('search-input');
  const list = document.getElementById('results-list');
  const status = document.getElementById('search-status');
  const btn = document.getElementById('search-button');
  const clearBtn = document.getElementById('clear-search');
  const returnLink = document.getElementById('return-link');

  // Handle Return Link
  if (document.referrer && document.referrer.includes(window.location.hostname) && !document.referrer.includes('/search')) {
    returnLink.href = document.referrer;
    returnLink.style.display = 'inline';
  }

  try {
    const response = await fetch("{{ '/search.json' | relative_url }}");
    const posts = await response.json();

    const performSearch = (queryOverride) => {
      const query = (queryOverride || input.value).toLowerCase().trim();
      input.value = query;
      list.innerHTML = '';
      clearBtn.style.display = query.length > 0 ? 'block' : 'none';

      if (query.length < 2) {
        status.textContent = query.length === 0 ? "" : "Please enter at least 2 characters.";
        return;
      }

      const results = posts.filter(post => 
        post.title.toLowerCase().includes(query) || 
        (post.tags && post.tags.some(t => t.toLowerCase().includes(query))) ||
        (post.categories && post.categories.some(c => c.toLowerCase().includes(query))) ||
        post.date.toLowerCase().includes(query)
      );

      if (results.length === 0) {
        status.textContent = `No results found for "${query}"`;
      } else {
        status.textContent = `Found ${results.length} article(s):`;
        results.forEach(post => {
          const li = document.createElement('li');
          li.style.padding = "15px 0";
          li.style.borderBottom = "1px solid #eee";
          li.innerHTML = `<a href="${post.url}" style="color: #b03931; font-weight: bold; text-decoration: none; font-size: 1.1rem;">${post.title}</a><br><small style="color: #888;">${post.date}</small>`;
          list.appendChild(li);
        });
      }
    };

    clearBtn.addEventListener('click', () => {
      input.value = '';
      list.innerHTML = '';
      status.textContent = '';
      clearBtn.style.display = 'none';
      input.focus();
    });

    const urlParams = new URLSearchParams(window.location.search);
    const initialQuery = urlParams.get('q');
    if (initialQuery) performSearch(initialQuery);

    input.addEventListener('input', () => performSearch());
    btn.addEventListener('click', () => performSearch());
    input.addEventListener('keypress', (e) => { if (e.key === 'Enter') performSearch(); });

  } catch (e) {
    status.textContent = "Error loading search index.";
  }
}
document.addEventListener('DOMContentLoaded', initSearch);
</script>