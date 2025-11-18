# JavaScript Extensions

## Web component

- 可將自定義HTML / CSS / JS的內容包成一個自定義Tag DOM，讓HTML看起來更乾淨

```Javascript
// Email Tag Web Component
class EmailTag extends HTMLElement {
  constructor() {
    super();
    
    // 取得 email 屬性
    const email = this.getAttribute('email') || '';
    
    // 建立簡單樣式和結構
    this.innerHTML = `
      <style>
        .email-tag {
          display: inline-block;
          border: 1px solid #ccc;
          padding: 2px 6px;
          cursor: pointer;
        }
      </style>
      <span class="email-tag">${email}</span>
    `;
    
    // 點擊開啟 Outlook
    this.querySelector('.email-tag').addEventListener('click', () => {
      window.location.href = `mailto:${email}`;
    });
  }
}

// 註冊自定義元素
customElements.define('email-tag', EmailTag);
```

- 使用範例

```html
<email-tag email="user@example.com"></email-tag>
<email-tag email="admin@company.com"></email-tag>
```

## CDN

- 加速第三方package，可用於減少專案大小
- 轉譯package給瀏覽器直接使用

### ESM.sh

- 可將NPM package直接作為ES Modules予瀏覽器使用而不需開發安裝
- 可轉譯TypeScript/Babel
- 壓縮及Minify
- 可自行下載特定版本為local js檔案，降低網路延遲

```HTML
<!-- Using diceBear avatars directly from ESM.sh -->
<script type="module">
  import { createAvatar } from 'https://esm.sh/@dicebear/core';
  import { lorelei } from 'https://esm.sh/@dicebear/lorelei';

  // Create avatar
  const avatar = createAvatar(lorelei, {
    seed: 'John',
    backgroundColor: ['b6e3f4','c0aede','d1d4f9']
  });

  // Convert to SVG string and display
  const svgString = avatar.toString();
  document.getElementById('avatar-container').innerHTML = svgString;
</script>

<div id="avatar-container"></div>
```

### ES Module

- defer: 可延遲執行
- isolation: 域不互相汙染
- CDN / Local來源支援: 使用NPM URL或地端的.js皆可
- importmap: Module來源命名

```HTML
<script type="importmap">
{
  "imports": {
    "lodash": "https://cdn.jsdelivr.net/npm/lodash-es@4.17.21/lodash.js"
  }
}
</script>

<script type="module">
  import _ from 'lodash';
  console.log(_.chunk([1,2,3,4], 2));
</script>
```