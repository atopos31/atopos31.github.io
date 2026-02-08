---
menus: main
title: 开源项目
---

<div class="open-source-list" data-stars>
  <div class="open-source-item" data-repo="atopos31/llmio">
    <h2><a href="https://github.com/atopos31/llmio">llmio</a> <span class="stars">☆ …</span></h2>
    <p class="desc" data-fallback="LLM API 负载均衡网关.">加载中…</p>
  </div>
  <div class="open-source-item" data-repo="nsxdevx/nsxbot">
    <h2><a href="https://github.com/nsxdevx/nsxbot">nsxbot</a> <span class="stars">☆ …</span></h2>
    <p class="desc" data-fallback="使用 Go (Golang) 语言编写，基于 OneBot11 协议的聊天机器人框架。">加载中…</p>
  </div>
  <div class="open-source-item" data-repo="atopos31/go-veilink">
    <h2><a href="https://github.com/atopos31/go-veilink">veilink</a> <span class="stars">☆ …</span></h2>
    <p class="desc" data-fallback="轻量级内网穿透工具。">加载中…</p>
  </div>
  <div class="open-source-item" data-repo="atopos31/freshpic">
    <h2><a href="https://github.com/atopos31/freshpic">freshpic</a> <span class="stars">☆ …</span></h2>
    <p class="desc" data-fallback="基于Fresh,Deno-kv,Deno-deploy实现免费图床。">加载中…</p>
  </div>
</div>

<script>
(function () {
  var container = document.querySelector('[data-stars]');
  if (!container) return;

  var items = container.querySelectorAll('[data-repo]');
  if (!items.length) return;

  items.forEach(function (item) {
    var repo = item.getAttribute('data-repo');
    var starsEl = item.querySelector('.stars');
    var descEl = item.querySelector('.desc');
    var fallbackDesc = descEl ? descEl.getAttribute('data-fallback') : '';
    if (!repo || !starsEl) return;

    fetch('https://api.github.com/repos/' + repo)
      .then(function (res) {
        if (!res.ok) throw new Error('bad response');
        return res.json();
      })
      .then(function (data) {
        var stars = data && typeof data.stargazers_count === 'number'
          ? data.stargazers_count
          : null;
        starsEl.textContent = stars === null ? '☆ —' : ('☆ ' + stars.toLocaleString());

        if (descEl) {
          var desc = data && typeof data.description === 'string' ? data.description.trim() : '';
          descEl.textContent = desc || fallbackDesc || '暂无简介';
        }
      })
      .catch(function () {
        starsEl.textContent = '☆ —';
        if (descEl) descEl.textContent = fallbackDesc || '暂无简介';
      });
  });
})();
</script>
