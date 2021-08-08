---
title: SvelteKit学习笔记
description: ''
date: 2021-08-07
img: https://res.cloudinary.com/neroblackstone/image/upload/v1625101749/svelte_y2yhr6.png
tags:
- SvelteKit

---
## HOOKS

一个可选的 `src/hooks.js`（或 `src/hooks.ts`，或 `src/hooks/index.js`）文件导出三个在服务器上运行的函数，都是可选的——**handle**、**getSession** 和 **serverFetch**。

### handle

函数针对页面和端点的每个请求运行，并确定响应。它接收`request`对象和一个名为 `resolve` 的函数，该函数调用 SvelteKit 的路由器并相应地生成响应。允许您修改响应标头或正文，或完全绕过 SvelteKit（例如，用于以编程方式实现端点）。

如果未实现，默认为 `({ request, resolve }) => resolve(request)`。

``` ts
// handle TypeScript type definitions

type Headers = Record<string, string>;

type Request<Locals = Record<string, any>> = {
	method: string;
	host: string;
	headers: Headers;
	path: string;
	params: Record<string, string>;
	query: URLSearchParams;
	rawBody: string | Uint8Array;
	body: ParameterizedBody<Body>;
	locals: Locals; // populated by hooks handle
};

type Response = {
	status: number;
	headers: Headers;
	body?: string | Uint8Array;
};

type Handle<Locals = Record<string, any>> = (input: {
	request: Request<Locals>;
	resolve: (request: Request<Locals>) => Response | Promise<Response>;
}) => Response | Promise<Response>;
```