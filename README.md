
# 使用 Jest 、 React Testing Library 、MSW 建立測試環境

    
### 主要套件
  
    Jest - 建立測試環境 提供 asseration 等功能判斷結果
    React Testing Library - 針對 React component 進行操作
    Mock Service Worker - MSW 攔截 API 作 mock 資料

    @jest/globals
    @testing-library/jest-dom
    @testing-library/react
    @types/jest
    jest
    jest-environment-jsdom - jsdom 環境 可測試 UI 的部分
    dotenv - 建立 env 環境參數
    ts-jest
    msw
    undici - msw 在 jsdom 下使用到 
    有雷!!!
    請使用 https://github.com/mswjs/examples/blob/main/examples/with-jest-jsdom/package.json msw example 中的版本
    現在直接install會是 6.多版會造成奇怪的錯誤

    如果需要使用 jsdom 且電腦是 mac M 系列
    需到此使用 homebrew 安裝相關套件
    https://github.com/Automattic/node-canvas

### 設定

這邊會以 jsdom 的測試環境為主

jest.config.ts

    // 這是 next.js 下的版本
    import type { Config } from 'jest'
    import nextJest from 'next/jest.js'
    
    const createJestConfig = nextJest({
      dir: './',
    })
     
    // Add any custom config to be passed to Jest
    const config: Config = {
      coverageProvider: 'v8',
      testEnvironment: 'jsdom',
      setupFiles: ['<rootDir>/jest.polyfills.ts'],
      setupFilesAfterEnv: ['<rootDir>/test/setup-tests.ts'],
      testEnvironmentOptions: {
        customExportConditions: [''],  // for msw
      },
    }
     
    export default createJestConfig(config)

jest.polyfills.ts  
這是根據 msw 來的  
https://mswjs.io/docs/migrations/1.x-to-2.x/  

    /**
     * @note The block below contains polyfills for Node.js globals
     * required for Jest to function when running JSDOM tests.
     * These HAVE to be require's and HAVE to be in this exact
     * order, since "undici" depends on the "TextEncoder" global API.
     *
     * Consider migrating to a more modern test runner if
     * you don't want to deal with this.
     */
    
    const { TextDecoder, TextEncoder,ReadableStream } = require('node:util')
    
    Object.defineProperties(globalThis, {
      TextDecoder: { value: TextDecoder },
      TextEncoder: { value: TextEncoder },
      ReadableStream: { value: ReadableStream },
    })
    
    const { Blob } = require('node:buffer')
    const { fetch, Headers, FormData, Request, Response } = require('undici')
    
    Object.defineProperties(globalThis, {
      fetch: { value: fetch, writable: true },
      Blob: { value: Blob },
      Headers: { value: Headers },
      FormData: { value: FormData },
      Request: { value: Request },
      Response: { value: Response },
    })


setup-tests.ts    
建立 env 中的參數    
建立 msw     
    
    import * as dotenv from 'dotenv';
    import { server } from './mocks/node';
    
    // dotenv config
    process.env.NAME = 'NAME';
    
    // 建立msw server
    beforeAll(() => {
      server.listen()
    })
    
    afterEach(() => {
      server.resetHandlers()
    })
    
    afterAll(() => {
      server.close()
    })

node.ts
建立 server

    import { setupServer } from 'msw/node'
    import { handlers } from './handlers'
    
    export const server = setupServer(...handlers)


handler.ts
設定 API response
url 需使用完整的網址 https://.....
只用 relative url 會攔截不到

網路上的範例有使用 import { rest } from 'msw'    
這是舊版的寫法 新版請使用 http


    import { http, HttpResponse } from 'msw'
    
    export const handlers = [
      http.post('url', () => HttpResponse.json({
    
      })
    )]
