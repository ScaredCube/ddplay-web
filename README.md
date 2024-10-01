# ddPlayer-web

网页端的弹幕播放器，基于 ArtPlayer 和 Dandanplay Api，支持iframe嵌入alist预览

## 直接使用

>  `https://ddplay-web.scc.moe/?videoUrl={视频链接}`

## Alist iframe 预览

```json
"mp4,MP4": {
    "Player with Danmuku":"https://ddplay-web.scc.moe/?videoUrl=$e_url"
}
```

## Project Setup

```sh
npm install
```

### Compile and Hot-Reload for Development

```sh
npm run dev
```

### Type-Check, Compile and Minify for Production

```sh
npm run build
```
