# Build Tools for TypeScript

This module covers popular build tools and bundlers used with TypeScript projects.

## Core Concepts

### 1. Webpack Configuration

```typescript
// webpack.config.ts
import path from 'path';
import webpack from 'webpack';
import HtmlWebpackPlugin from 'html-webpack-plugin';
import MiniCssExtractPlugin from 'mini-css-extract-plugin';

const config: webpack.Configuration = {
  entry: './src/index.ts',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[contenthash].js'
  },
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: 'ts-loader',
        exclude: /node_modules/
      },
      {
        test: /\.css$/,
        use: [MiniCssExtractPlugin.loader, 'css-loader']
      }
    ]
  },
  resolve: {
    extensions: ['.tsx', '.ts', '.js'],
    alias: {
      '@': path.resolve(__dirname, 'src')
    }
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: './src/index.html'
    }),
    new MiniCssExtractPlugin({
      filename: '[name].[contenthash].css'
    })
  ],
  optimization: {
    splitChunks: {
      chunks: 'all'
    }
  }
};

export default config;
```

### 2. Vite Configuration

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import tsconfigPaths from 'vite-tsconfig-paths';

export default defineConfig({
  plugins: [react(), tsconfigPaths()],
  resolve: {
    alias: {
      '@': '/src'
    }
  },
  build: {
    target: 'esnext',
    sourcemap: true,
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          utils: ['lodash', 'date-fns']
        }
      }
    }
  }
});
```

### 3. ESBuild Configuration

```typescript
// esbuild.config.ts
import { build } from 'esbuild';
import { nodeExternalsPlugin } from 'esbuild-node-externals';

build({
  entryPoints: ['src/index.ts'],
  outdir: 'dist',
  bundle: true,
  minify: true,
  platform: 'node',
  target: 'node14',
  format: 'esm',
  sourcemap: true,
  plugins: [nodeExternalsPlugin()],
  define: {
    'process.env.NODE_ENV': '"production"'
  }
}).catch(() => process.exit(1));
```

## Real-World Use Cases

### 1. Full-Stack TypeScript Project

```typescript
// build.config.ts
import { build } from 'esbuild';
import { copy } from 'fs-extra';

async function buildProject() {
  // Build server
  await build({
    entryPoints: ['src/server/index.ts'],
    outdir: 'dist/server',
    platform: 'node',
    format: 'cjs',
    target: 'node14',
    sourcemap: true
  });

  // Build client
  await build({
    entryPoints: ['src/client/index.tsx'],
    outdir: 'dist/client',
    platform: 'browser',
    format: 'esm',
    bundle: true,
    minify: true,
    sourcemap: true,
    loader: {
      '.png': 'dataurl',
      '.svg': 'text'
    }
  });

  // Copy static assets
  await copy('src/public', 'dist/public');
}

buildProject().catch(console.error);
```

### 2. Library Build Pipeline

```typescript
// rollup.config.ts
import { defineConfig } from 'rollup';
import typescript from '@rollup/plugin-typescript';
import dts from 'rollup-plugin-dts';
import { terser } from 'rollup-plugin-terser';

export default defineConfig([
  {
    input: 'src/index.ts',
    output: [
      {
        file: 'dist/index.js',
        format: 'cjs'
      },
      {
        file: 'dist/index.esm.js',
        format: 'esm'
      },
      {
        file: 'dist/index.min.js',
        format: 'umd',
        name: 'MyLibrary',
        plugins: [terser()]
      }
    ],
    plugins: [
      typescript({
        tsconfig: './tsconfig.json',
        declaration: true,
        declarationDir: 'dist/types'
      })
    ],
    external: ['react', 'react-dom']
  },
  {
    input: 'dist/types/index.d.ts',
    output: [{ file: 'dist/index.d.ts', format: 'es' }],
    plugins: [dts()]
  }
]);
```

## Mini-Project: Custom Build Tool

```typescript
// builder.ts
import { build, BuildOptions } from 'esbuild';
import { watch } from 'chokidar';
import { debounce } from 'lodash';

interface BuilderConfig {
  entryPoints: string[];
  outdir: string;
  watch?: boolean;
  minify?: boolean;
  sourcemap?: boolean;
}

class Builder {
  private config: BuilderConfig;
  private buildOptions: BuildOptions;

  constructor(config: BuilderConfig) {
    this.config = config;
    this.buildOptions = {
      entryPoints: config.entryPoints,
      outdir: config.outdir,
      bundle: true,
      minify: config.minify,
      sourcemap: config.sourcemap,
      platform: 'browser',
      target: 'es2020',
      format: 'esm'
    };
  }

  async build(): Promise<void> {
    console.time('Build completed in');

    try {
      await build(this.buildOptions);
      console.timeEnd('Build completed in');
    } catch (error) {
      console.error('Build failed:', error);
      process.exit(1);
    }
  }

  watch(): void {
    if (!this.config.watch) return;

    const rebuild = debounce(async () => {
      console.log('Rebuilding...');
      await this.build();
    }, 100);

    watch(this.config.entryPoints, {
      persistent: true
    }).on('change', rebuild);

    console.log('Watching for changes...');
  }
}

// Usage
const builder = new Builder({
  entryPoints: ['src/index.ts'],
  outdir: 'dist',
  watch: true,
  minify: true,
  sourcemap: true
});

builder.build().then(() => builder.watch());
```

## Best Practices

1. Use the right tool for the job
2. Optimize build performance
3. Generate source maps in development
4. Split code appropriately
5. Implement caching strategies

## Common Mistakes

### ❌ Bad Practice: Inefficient Build Configuration

```typescript
// Bad: No code splitting or optimization
const config = {
  entry: './src/index.ts',
  output: {
    filename: 'bundle.js'
  }
};
```

### ✅ Good Practice: Optimized Build Configuration

```typescript
// Good: Proper code splitting and optimization
const config = {
  entry: './src/index.ts',
  output: {
    filename: '[name].[contenthash].js',
    chunkFilename: '[name].[contenthash].chunk.js'
  },
  optimization: {
    moduleIds: 'deterministic',
    runtimeChunk: 'single',
    splitChunks: {
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all'
        }
      }
    }
  }
};
```

### ❌ Bad Practice: No Development Optimizations

```typescript
// Bad: Same configuration for all environments
export default {
  minify: true,
  sourcemap: false
};
```

### ✅ Good Practice: Environment-Specific Configuration

```typescript
// Good: Environment-specific settings
export default defineConfig(({ mode }) => ({
  minify: mode === 'production',
  sourcemap: mode === 'development',
  define: {
    'process.env.NODE_ENV': `"${mode}"`
  }
}));
```

## Quiz

1. Which bundler is known for its speed?
   - a) Webpack
   - b) Rollup
   - c) esbuild
   - d) Parcel

2. When should you use code splitting?
   - a) Never
   - b) Always
   - c) Only in production
   - d) When it improves loading performance

3. What's the purpose of source maps?
   - a) Reduce bundle size
   - b) Debug production code
   - c) Debug transformed code
   - d) Improve performance

4. Which tool is best for library builds?
   - a) Webpack
   - b) Rollup
   - c) esbuild
   - d) Parcel

Answers: 1-c, 2-d, 3-c, 4-b

## Recap

- Different build tools serve different purposes
- Webpack for complex applications
- Vite for modern development
- esbuild for performance
- Rollup for libraries
- Optimize builds for development and production

---
**Navigation**

[Previous: TypeScript Compiler](51-typescript-compiler.md) | [Next: Debug Tools](53-debug-tools.md)