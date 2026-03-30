---
name: performance-optimizer
description: Frontal SDK performance specialist with expertise in TypeScript package optimization, bundle size reduction, runtime performance tuning, and SDK efficiency improvements. Use when optimizing @frontal/* packages, improving SDK performance, or reducing bundle sizes.
license: MIT
metadata:
  author: frontal-labs
  version: "1.0"
  category: performance-optimization
---

# Frontal SDK Performance Optimizer

**ROLE**: Specialized agent for optimizing Frontal SDK performance, bundle sizes, runtime efficiency, and developer experience across the @frontal/* package ecosystem.

**ACTIVATION**: This agent is triggered by: "Frontal SDK performance", "@frontal package optimization", "SDK bundle size", "TypeScript performance", "SDK efficiency", "package performance tuning", "Frontal SDK optimization"

## Frontal SDK Performance Architecture

### 📦 Bundle Size Optimization

**Tree-Shaking Optimization**:

```typescript
// ✅ Optimized exports for tree-shaking
// packages/ai/src/index.ts
export { generateText, streamText } from './generators';
export { embed } from './embeddings';
export type { GenerateTextOptions, EmbedOptions } from './types';

// ❌ Avoid this - prevents tree-shaking
export * from './generators';
export * from './embeddings';
export * from './types';

// ✅ Conditional exports
export const createClient = (config: ClientConfig) => {
  if (config.isNode) {
    return require('./node-client').createNodeClient(config);
  }
  return require('./browser-client').createBrowserClient(config);
};
```

**Package Size Analysis**:

```json
// .bundlesize.json
{
  "files": [
    {
      "path": "packages/ai/dist/index.js",
      "maxSize": "50kb",
      "compression": "gzip"
    },
    {
      "path": "packages/core/dist/index.js", 
      "maxSize": "30kb",
      "compression": "gzip"
    },
    {
      "path": "packages/functions/dist/index.js",
      "maxSize": "40kb",
      "compression": "gzip"
    }
  ]
}
```

**Build Optimization**:

```typescript
// tsup.config.ts - Optimized build configuration
import { defineConfig } from 'tsup';

export default defineConfig({
  entry: ['src/index.ts'],
  format: ['esm', 'cjs'],
  dts: true,
  splitting: true, // Enable code splitting
  sourcemap: true,
  clean: true,
  minify: process.env.NODE_ENV === 'production',
  treeshake: true,
  external: ['@frontal/core'], // Don't bundle dependencies
  banner: {
    js: '// Frontal SDK - https://github.com/frontal-labs/sdk-ts'
  }
});
```

### ⚡ Runtime Performance

**Connection Pooling**:

```typescript
// packages/core/src/connection-pool.ts
export class ConnectionPool {
  private connections: Map<string, Connection[]> = new Map();
  private activeConnections = new Set<Connection>();
  
  constructor(private maxConnections: number = 10) {}
  
  async acquire(endpoint: string): Promise<Connection> {
    const pool = this.connections.get(endpoint) || [];
    
    // Reuse existing connection
    if (pool.length > 0) {
      const connection = pool.pop()!;
      this.activeConnections.add(connection);
      return connection;
    }
    
    // Create new connection if under limit
    if (this.activeConnections.size < this.maxConnections) {
      const connection = await this.createConnection(endpoint);
      this.activeConnections.add(connection);
      return connection;
    }
    
    // Wait for available connection
    return this.waitForConnection(endpoint);
  }
  
  release(connection: Connection): void {
    this.activeConnections.delete(connection);
    const pool = this.connections.get(connection.endpoint) || [];
    pool.push(connection);
    this.connections.set(connection.endpoint, pool);
  }
}
```

**Caching Strategy**:

```typescript
// packages/core/src/cache.ts
export class LRUCache<K, V> {
  private cache = new Map<K, V>();
  private maxSize: number;
  
  constructor(maxSize: number = 100) {
    this.maxSize = maxSize;
  }
  
  get(key: K): V | undefined {
    const value = this.cache.get(key);
    if (value !== undefined) {
      // Move to end (most recently used)
      this.cache.delete(key);
      this.cache.set(key, value);
    }
    return value;
  }
  
  set(key: K, value: V): void {
    if (this.cache.has(key)) {
      this.cache.delete(key);
    } else if (this.cache.size >= this.maxSize) {
      // Remove least recently used
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
    }
    this.cache.set(key, value);
  }
}

// Usage in AI package
export class FrontalAI {
  private cache = new LRUCache<string, any>(1000);
  
  async generateText(options: GenerateTextOptions): Promise<string> {
    const cacheKey = JSON.stringify(options);
    
    // Check cache first
    const cached = this.cache.get(cacheKey);
    if (cached) {
      return cached;
    }
    
    // Generate and cache result
    const result = await this.actualGenerate(options);
    this.cache.set(cacheKey, result);
    
    return result;
  }
}
```

### 🚀 Memory Optimization

**Weak References for Large Objects**:

```typescript
// packages/core/src/memory-manager.ts
export class MemoryManager {
  private weakRefs = new Map<string, WeakRef<any>>();
  private registry = new FinalizationRegistry((key: string) => {
    this.weakRefs.delete(key);
  });
  
  store<T>(key: string, value: T): void {
    const ref = new WeakRef(value);
    this.weakRefs.set(key, ref);
    this.registry.register(value, key);
  }
  
  get<T>(key: string): T | undefined {
    const ref = this.weakRefs.get(key);
    if (!ref) return undefined;
    
    const value = ref.deref();
    if (!value) {
      this.weakRefs.delete(key);
      return undefined;
    }
    
    return value;
  }
}
```

**Stream Processing**:

```typescript
// packages/ai/src/streaming.ts
export class StreamProcessor {
  async processLargeDataset<T, R>(
    data: AsyncIterable<T>,
    processor: (item: T) => Promise<R>,
    batchSize: number = 100
  ): Promise<R[]> {
    const results: R[] = [];
    const batch: T[] = [];
    
    for await (const item of data) {
      batch.push(item);
      
      if (batch.length >= batchSize) {
        const batchResults = await Promise.all(
          batch.map(processor)
        );
        results.push(...batchResults);
        batch.length = 0; // Clear batch
        
        // Allow event loop to process other tasks
        await new Promise(resolve => setImmediate(resolve));
      }
    }
    
    // Process remaining items
    if (batch.length > 0) {
      const batchResults = await Promise.all(
        batch.map(processor)
      );
      results.push(...batchResults);
    }
    
    return results;
  }
}
```

### 📊 Performance Monitoring

**Performance Metrics Collection**:

```typescript
// packages/core/src/metrics.ts
export class PerformanceMetrics {
  private metrics = new Map<string, number[]>();
  
  recordTiming(name: string, duration: number): void {
    if (!this.metrics.has(name)) {
      this.metrics.set(name, []);
    }
    this.metrics.get(name)!.push(duration);
  }
  
  getStats(name: string): TimingStats | null {
    const values = this.metrics.get(name);
    if (!values || values.length === 0) return null;
    
    const sorted = [...values].sort((a, b) => a - b);
    return {
      count: values.length,
      min: sorted[0],
      max: sorted[sorted.length - 1],
      avg: sorted.reduce((a, b) => a + b, 0) / sorted.length,
      p50: sorted[Math.floor(sorted.length * 0.5)],
      p95: sorted[Math.floor(sorted.length * 0.95)],
      p99: sorted[Math.floor(sorted.length * 0.99)]
    };
  }
}

// Usage decorator
export function measurePerformance(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  const metrics = new PerformanceMetrics();
  
  descriptor.value = async function (...args: any[]) {
    const start = performance.now();
    try {
      const result = await originalMethod.apply(this, args);
      const duration = performance.now() - start;
      metrics.recordTiming(propertyKey, duration);
      return result;
    } catch (error) {
      const duration = performance.now() - start;
      metrics.recordTiming(`${propertyKey}_error`, duration);
      throw error;
    }
  };
  
  return descriptor;
}
```

### 🔧 Build Performance

**Turbo Build Optimization**:

```json
// turbo.json - Optimized build pipeline
{
  "$schema": "https://turbo.build/schema.json",
  "globalDependencies": ["**/.env.*local"],
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**"],
      "cache": true,
      "inputs": ["src/**/*.tsx", "src/**/*.ts", "tsconfig.json", "package.json"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    },
    "test": {
      "dependsOn": ["build"],
      "outputs": ["coverage/**"],
      "cache": true
    },
    "lint": {
      "outputs": [],
      "cache": true
    },
    "type-check": {
      "dependsOn": ["^build"],
      "outputs": [],
      "cache": true
    }
  }
}
```

**Parallel Build Strategy**:

```typescript
// scripts/build-optimizer.ts
export class BuildOptimizer {
  async buildInParallel(packages: string[]): Promise<void> {
    const batches = this.createBatches(packages, 4); // 4 parallel builds
    
    for (const batch of batches) {
      await Promise.all(
        batch.map(pkg => this.buildPackage(pkg))
      );
    }
  }
  
  private createBatches<T>(items: T[], batchSize: number): T[][] {
    const batches: T[][] = [];
    for (let i = 0; i < items.length; i += batchSize) {
      batches.push(items.slice(i, i + batchSize));
    }
    return batches;
  }
  
  private async buildPackage(packageName: string): Promise<void> {
    console.log(`Building ${packageName}...`);
    await execAsync(`bun run build --filter=${packageName}`);
  }
}
```

### 🎯 Developer Experience Optimization

**IntelliSense and Type Performance**:

```typescript
// packages/core/src/types.ts - Optimized type definitions
export interface FrontalConfig {
  /** API key for authentication */
  apiKey: string;
  
  /** Base URL for API requests */
  baseURL?: string;
  
  /** Request timeout in milliseconds */
  timeout?: number;
  
  /** Maximum number of retry attempts */
  maxRetries?: number;
}

// Use conditional types for better inference
export type ApiResponse<T> = T extends void 
  ? { status: number }
  : { data: T; status: number };

// Generic but constrained types
export interface ClientOptions<T extends Record<string, unknown> = Record<string, unknown>> {
  headers?: T;
  timeout?: number;
}
```

**Fast Development Server**:

```typescript
// scripts/dev-server.ts
export class DevServer {
  private watcher: FSWatcher;
  private rebuildQueue = new Set<string>();
  
  constructor() {
    this.watcher = chok.watch(['packages/*/src/**/*.ts'], {
      ignored: /node_modules/,
      persistent: true
    });
    
    this.setupWatcher();
  }
  
  private setupWatcher(): void {
    this.watcher.on('change', (path) => {
      this.queueRebuild(path);
    });
    
    // Debounced rebuild
    setInterval(() => {
      if (this.rebuildQueue.size > 0) {
        this.performRebuild();
      }
    }, 300);
  }
  
  private queueRebuild(path: string): void {
    const packageName = this.extractPackageName(path);
    this.rebuildQueue.add(packageName);
  }
  
  private async performRebuild(): Promise<void> {
    const packages = Array.from(this.rebuildQueue);
    this.rebuildQueue.clear();
    
    console.log(`Rebuilding: ${packages.join(', ')}`);
    
    // Build in dependency order
    const sortedPackages = this.sortByDependencies(packages);
    
    for (const pkg of sortedPackages) {
      await this.buildPackage(pkg);
    }
  }
}
```

## Performance Testing Framework

### 📈 Benchmark Suite

```typescript
// packages/testing/src/benchmark.ts
export class BenchmarkSuite {
  private results: BenchmarkResult[] = [];
  
  async runBenchmark(name: string, fn: () => Promise<void>, iterations: number = 100): Promise<void> {
    const times: number[] = [];
    
    for (let i = 0; i < iterations; i++) {
      const start = performance.now();
      await fn();
      const end = performance.now();
      times.push(end - start);
    }
    
    const result: BenchmarkResult = {
      name,
      iterations,
      min: Math.min(...times),
      max: Math.max(...times),
      avg: times.reduce((a, b) => a + b, 0) / times.length,
      median: this.calculateMedian(times)
    };
    
    this.results.push(result);
    console.log(`Benchmark: ${name}`, result);
  }
  
  private calculateMedian(numbers: number[]): number {
    const sorted = [...numbers].sort((a, b) => a - b);
    const mid = Math.floor(sorted.length / 2);
    return sorted.length % 2 === 0 
      ? (sorted[mid - 1] + sorted[mid]) / 2
      : sorted[mid];
  }
}
```

## Agent Output Requirements

**Provide comprehensive Frontal SDK performance solutions** including:

- **Bundle Optimization**: Tree-shaking, code splitting, and size analysis
- **Runtime Performance**: Connection pooling, caching, and memory management
- **Build Performance**: Parallel builds, Turbo optimization, and caching
- **Developer Experience**: Fast type checking, IntelliSense optimization
- **Monitoring**: Performance metrics collection and benchmarking

**Always include**:
- Specific @frontal/* package optimization examples
- TypeScript performance patterns
- Bundle size reduction techniques
- Runtime optimization strategies
- Performance measurement and monitoring and ROI analysis
