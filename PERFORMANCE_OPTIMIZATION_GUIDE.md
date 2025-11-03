# Performance Optimization Guide 🚀

A comprehensive guide to identifying and improving slow or inefficient code across different tech stacks.

## Table of Contents
- [Web Development](#web-development)
  - [React Optimization](#react-optimization)
  - [Angular Optimization](#angular-optimization)
  - [JavaScript/TypeScript General](#javascripttypescript-general)
  - [CSS/SCSS Optimization](#cssscss-optimization)
- [Python & Data Science](#python--data-science)
- [General Performance Tools](#general-performance-tools)
- [Best Practices](#best-practices)

---

## Web Development

### React Optimization

#### 1. **Avoid Unnecessary Re-renders**
```javascript
// ❌ Bad: Component re-renders on every parent update
const UserProfile = ({ user, settings }) => {
  return <div>{user.name}</div>;
};

// ✅ Good: Use React.memo to prevent unnecessary re-renders
const UserProfile = React.memo(({ user, settings }) => {
  return <div>{user.name}</div>;
}, (prevProps, nextProps) => {
  return prevProps.user.id === nextProps.user.id;
});
```

#### 2. **Use useMemo and useCallback**
```javascript
// ❌ Bad: Expensive calculation runs on every render
const Component = ({ items }) => {
  const sortedItems = items.sort((a, b) => a.value - b.value);
  return <List items={sortedItems} />;
};

// ✅ Good: Memoize expensive calculations
const Component = ({ items }) => {
  const sortedItems = useMemo(
    () => items.sort((a, b) => a.value - b.value),
    [items]
  );
  return <List items={sortedItems} />;
};
```

#### 3. **Lazy Loading and Code Splitting**
```javascript
// ❌ Bad: Load all components upfront
import Dashboard from './Dashboard';
import Settings from './Settings';
import Profile from './Profile';

// ✅ Good: Lazy load components
const Dashboard = lazy(() => import('./Dashboard'));
const Settings = lazy(() => import('./Settings'));
const Profile = lazy(() => import('./Profile'));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/settings" element={<Settings />} />
        <Route path="/profile" element={<Profile />} />
      </Routes>
    </Suspense>
  );
}
```

#### 4. **Virtual Scrolling for Large Lists**
```javascript
// ❌ Bad: Render all 10,000 items
const List = ({ items }) => (
  <div>
    {items.map(item => <Item key={item.id} {...item} />)}
  </div>
);

// ✅ Good: Use virtual scrolling (react-window or react-virtualized)
import { FixedSizeList } from 'react-window';

const List = ({ items }) => (
  <FixedSizeList
    height={600}
    itemCount={items.length}
    itemSize={50}
  >
    {({ index, style }) => (
      <div style={style}>
        <Item {...items[index]} />
      </div>
    )}
  </FixedSizeList>
);
```

#### 5. **Optimize State Management**
```javascript
// ❌ Bad: Storing derived state
const [users, setUsers] = useState([]);
const [userCount, setUserCount] = useState(0);

// ✅ Good: Calculate derived values
const [users, setUsers] = useState([]);
const userCount = users.length;
```

### Angular Optimization

#### 1. **OnPush Change Detection**
```typescript
// ❌ Bad: Default change detection runs frequently
@Component({
  selector: 'app-user-list',
  templateUrl: './user-list.component.html'
})
export class UserListComponent {
  @Input() users: User[];
}

// ✅ Good: Use OnPush for better performance
@Component({
  selector: 'app-user-list',
  templateUrl: './user-list.component.html',
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class UserListComponent {
  @Input() users: User[];
}
```

#### 2. **TrackBy in *ngFor**
```typescript
// ❌ Bad: Angular re-creates DOM nodes unnecessarily
<div *ngFor="let item of items">
  {{ item.name }}
</div>

// ✅ Good: Use trackBy to identify items
<div *ngFor="let item of items; trackBy: trackByFn">
  {{ item.name }}
</div>

trackByFn(index: number, item: Item): number {
  return item.id;
}
```

#### 3. **Lazy Loading Modules**
```typescript
// ✅ Good: Lazy load feature modules
const routes: Routes = [
  {
    path: 'dashboard',
    loadChildren: () => import('./dashboard/dashboard.module').then(m => m.DashboardModule)
  },
  {
    path: 'settings',
    loadChildren: () => import('./settings/settings.module').then(m => m.SettingsModule)
  }
];
```

#### 4. **Unsubscribe from Observables**
```typescript
// ❌ Bad: Memory leak from unsubscribed observable
export class Component implements OnInit {
  ngOnInit() {
    this.userService.getUsers().subscribe(users => {
      this.users = users;
    });
  }
}

// ✅ Good: Clean up subscriptions
export class Component implements OnInit, OnDestroy {
  private destroy$ = new Subject<void>();
  
  ngOnInit() {
    this.userService.getUsers()
      .pipe(takeUntil(this.destroy$))
      .subscribe(users => {
        this.users = users;
      });
  }
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

#### 5. **Pure Pipes**
```typescript
// ✅ Good: Use pure pipes for transformations
@Pipe({
  name: 'filter',
  pure: true  // Default, but explicit is better
})
export class FilterPipe implements PipeTransform {
  transform(items: any[], searchTerm: string): any[] {
    if (!searchTerm) return items;
    return items.filter(item => 
      item.name.toLowerCase().includes(searchTerm.toLowerCase())
    );
  }
}
```

### JavaScript/TypeScript General

#### 1. **Debouncing and Throttling**
```javascript
// ❌ Bad: Function fires on every keystroke
input.addEventListener('keyup', (e) => {
  searchAPI(e.target.value);
});

// ✅ Good: Debounce expensive operations
const debounce = (func, delay) => {
  let timeoutId;
  return (...args) => {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => func(...args), delay);
  };
};

input.addEventListener('keyup', debounce((e) => {
  searchAPI(e.target.value);
}, 300));
```

#### 2. **Avoid Blocking the Main Thread**
```javascript
// ❌ Bad: Heavy computation blocks UI
function processLargeDataset(data) {
  return data.map(item => expensiveOperation(item));
}

// ✅ Good: Use Web Workers for heavy computations
// main.js
const worker = new Worker('worker.js');
worker.postMessage(data);
worker.onmessage = (e) => {
  console.log('Processed data:', e.data);
};

// worker.js
self.onmessage = (e) => {
  const result = e.data.map(item => expensiveOperation(item));
  self.postMessage(result);
};
```

#### 3. **Efficient Array Operations**
```javascript
// ❌ Bad: Multiple iterations over the same array
const filtered = users.filter(u => u.active);
const mapped = filtered.map(u => u.name);
const sorted = mapped.sort();

// ✅ Good: Single iteration with reduce
const result = users.reduce((acc, user) => {
  if (user.active) {
    acc.push(user.name);
  }
  return acc;
}, []).sort();
```

#### 4. **Optimize Object/Array Lookup**
```javascript
// ❌ Bad: O(n) lookup in array
function findUser(userId) {
  return users.find(u => u.id === userId);
}

// ✅ Good: O(1) lookup with Map
const userMap = new Map(users.map(u => [u.id, u]));
function findUser(userId) {
  return userMap.get(userId);
}
```

#### 5. **Async/Await for Parallel Operations**
```javascript
// ❌ Bad: Sequential async operations
async function getData() {
  const users = await fetchUsers();
  const posts = await fetchPosts();
  const comments = await fetchComments();
  return { users, posts, comments };
}

// ✅ Good: Parallel async operations
async function getData() {
  const [users, posts, comments] = await Promise.all([
    fetchUsers(),
    fetchPosts(),
    fetchComments()
  ]);
  return { users, posts, comments };
}
```

### CSS/SCSS Optimization

#### 1. **Avoid Expensive CSS Properties**
```scss
// ❌ Bad: Triggers layout recalculation
.element {
  width: 100px;
  height: 100px;
  transition: width 0.3s;
}

// ✅ Good: Use transform (composited properties)
.element {
  width: 100px;
  height: 100px;
  transform: scaleX(1);
  transition: transform 0.3s;
}
```

#### 2. **Use CSS Containment**
```scss
// ✅ Good: Limit layout calculations
.card {
  contain: layout style paint;
}

.sidebar {
  contain: layout;
}
```

#### 3. **Optimize Selectors**
```scss
// ❌ Bad: Overly specific and slow selectors
body div.container ul li a.link { }

// ✅ Good: Simple, specific selectors
.nav-link { }
```

#### 4. **Use will-change Sparingly**
```scss
// ❌ Bad: Overuse of will-change
.element {
  will-change: transform, opacity, width, height;
}

// ✅ Good: Only for elements about to animate
.modal.is-opening {
  will-change: transform, opacity;
}
.modal.is-open {
  will-change: auto;
}
```

---

## Python & Data Science

### 1. **Use NumPy for Numerical Operations**
```python
# ❌ Bad: Pure Python loops
def sum_squares(n):
    total = 0
    for i in range(n):
        total += i ** 2
    return total

# ✅ Good: Vectorized NumPy operations
import numpy as np

def sum_squares(n):
    return np.sum(np.arange(n) ** 2)
```

### 2. **Pandas Performance**
```python
# ❌ Bad: Iterating over DataFrame rows
total = 0
for index, row in df.iterrows():
    total += row['value'] * row['multiplier']

# ✅ Good: Vectorized operations
total = (df['value'] * df['multiplier']).sum()
```

### 3. **Use List Comprehensions**
```python
# ❌ Bad: Loop with append
squares = []
for i in range(1000):
    squares.append(i ** 2)

# ✅ Good: List comprehension
squares = [i ** 2 for i in range(1000)]

# ✅ Even Better: Generator for large datasets
squares = (i ** 2 for i in range(1000000))
```

### 4. **Caching with functools**
```python
# ❌ Bad: Recalculating expensive operations
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

# ✅ Good: Cache results
from functools import lru_cache

@lru_cache(maxsize=None)
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)
```

### 5. **Use Built-in Functions**
```python
# ❌ Bad: Manual implementation
def get_max(numbers):
    maximum = numbers[0]
    for num in numbers:
        if num > maximum:
            maximum = num
    return maximum

# ✅ Good: Use built-ins (implemented in C)
maximum = max(numbers)
```

### 6. **Efficient Data Loading**
```python
# ❌ Bad: Loading entire dataset into memory
df = pd.read_csv('large_file.csv')

# ✅ Good: Use chunking for large files
chunk_size = 10000
for chunk in pd.read_csv('large_file.csv', chunksize=chunk_size):
    process_chunk(chunk)
```

### 7. **Multiprocessing for CPU-Bound Tasks**
```python
# ❌ Bad: Sequential processing
results = [expensive_function(item) for item in items]

# ✅ Good: Parallel processing
from multiprocessing import Pool

with Pool() as pool:
    results = pool.map(expensive_function, items)
```

### 8. **Use Sets for Membership Testing**
```python
# ❌ Bad: O(n) lookup in list
valid_ids = [1, 2, 3, 4, 5, ...]  # Large list
if user_id in valid_ids:  # Slow
    pass

# ✅ Good: O(1) lookup in set
valid_ids = {1, 2, 3, 4, 5, ...}
if user_id in valid_ids:  # Fast
    pass
```

---

## General Performance Tools

### Profiling Tools

#### JavaScript/Node.js
- **Chrome DevTools Performance Tab**: Profile browser performance
- **Lighthouse**: Audit web app performance
- **webpack-bundle-analyzer**: Analyze bundle sizes
- **clinic.js**: Profile Node.js applications

#### Python
- **cProfile**: Built-in profiler
```python
import cProfile
cProfile.run('your_function()')
```

- **line_profiler**: Line-by-line profiling
```python
@profile
def slow_function():
    # Your code here
```

- **memory_profiler**: Monitor memory usage
```python
from memory_profiler import profile

@profile
def memory_intensive_function():
    # Your code here
```

#### General
- **Browser DevTools Network Tab**: Analyze network requests
- **React DevTools Profiler**: Profile React component renders
- **Angular DevTools**: Debug Angular applications

---

## Best Practices

### 1. **Measure Before Optimizing**
- Always profile your code before optimizing
- Focus on actual bottlenecks, not perceived ones
- Use real-world data and scenarios

### 2. **Database Optimization**
```sql
-- ❌ Bad: N+1 query problem
-- ✅ Good: Use JOINs or eager loading

-- Add indexes on frequently queried columns
CREATE INDEX idx_user_email ON users(email);

-- Use EXPLAIN to analyze query performance
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';
```

### 3. **Caching Strategies**
- Browser caching (HTTP headers)
- In-memory caching (Redis, Memcached)
- CDN for static assets
- API response caching

### 4. **Lazy Loading Assets**
```html
<!-- ✅ Good: Lazy load images -->
<img src="placeholder.jpg" data-src="actual-image.jpg" loading="lazy" alt="Description">
```

### 5. **Minimize Bundle Size**
- Tree-shaking unused code
- Code splitting
- Compress assets (gzip, brotli)
- Use production builds

### 6. **Network Optimization**
- HTTP/2 or HTTP/3
- Minimize HTTP requests
- Use WebSockets for real-time data
- Implement request batching

### 7. **Monitor Performance**
- Set up performance monitoring (New Relic, Datadog, etc.)
- Track key metrics:
  - First Contentful Paint (FCP)
  - Time to Interactive (TTI)
  - Largest Contentful Paint (LCP)
  - Cumulative Layout Shift (CLS)
  - First Input Delay (FID)

---

## Quick Performance Checklist

### Before Deployment
- [ ] Run performance profiling tools
- [ ] Check bundle sizes
- [ ] Test with production data volumes
- [ ] Verify no memory leaks
- [ ] Test on slow networks (throttling)
- [ ] Verify mobile performance
- [ ] Check Core Web Vitals scores
- [ ] Review database query performance
- [ ] Implement appropriate caching
- [ ] Compress and optimize images

### During Development
- [ ] Use React.memo/Angular OnPush when appropriate
- [ ] Implement virtual scrolling for large lists
- [ ] Lazy load routes and components
- [ ] Debounce/throttle event handlers
- [ ] Use efficient data structures
- [ ] Avoid premature optimization
- [ ] Write performance tests

---

## Resources

### Tools
- [Chrome DevTools](https://developer.chrome.com/docs/devtools/)
- [Lighthouse](https://developers.google.com/web/tools/lighthouse)
- [WebPageTest](https://www.webpagetest.org/)
- [React DevTools](https://react.dev/learn/react-developer-tools)
- [Angular DevTools](https://angular.io/guide/devtools)

### Documentation
- [Web.dev Performance](https://web.dev/performance/)
- [React Performance Optimization](https://react.dev/learn/render-and-commit)
- [Angular Performance](https://angular.io/guide/performance-best-practices)
- [Python Performance Tips](https://wiki.python.org/moin/PythonSpeed/PerformanceTips)

---

**Remember**: "Premature optimization is the root of all evil" - Donald Knuth

Focus on writing clean, maintainable code first, then optimize based on measured performance bottlenecks. 🚀
