# Describe
Project to cheat studio-treed-ai job

# Script
## Open tools 
```
window.open(`https://treed-rbmc.onrender.com/?user=${document.querySelector('.tc-333').innerText.trim()}&projectId=${new URLSearchParams(window.location.search.substring(1)).get("projectId")}&taskId=${new URLSearchParams(window.location.search.substring(1)).get("taskId")}&authorization=` + localStorage.getItem("TOKEN_AUTHORIZATION").replaceAll("%22", ""), '_blank');
```

## Remove watermark
```
document.querySelectorAll('.watermark-group').forEach(e => e.remove());
```

## Open menu on top
```
var style = document.createElement('style');
style.type = 'text/css';
style.innerHTML = '.xmain {display: flex;position: fixed;top: 50px;vertical-align: top;left: 300px;width: 100%;background-color: lightblue;} ';
document.getElementsByTagName('head')[0].appendChild(style);
document.querySelectorAll('data-v-0fcb1dd3]')[3].className = 'xmain';
```

# Project Setup

```sh
yarn install
```

```sh
yarn dev
```
