 
## **`useFetch` Custom Hook**

### **Functionalities and Expectations**

A robust `useFetch` hook should handle data fetching efficiently and provide a clean and reusable interface for various components. Here are the key functionalities and areas to evaluate:

#### **a. Core Fetching Capabilities**

- **Asynchronous Data Fetching:**
  - Ability to perform GET, POST, PUT, DELETE requests as needed.
  - Support for fetching data from RESTful APIs or other data sources.

- **Request Configuration:**
  - Allow customization of request parameters such as headers, method, body, etc.
  - Support for query parameters and dynamic URLs.

#### **b. State Management**

- **Loading State:**
  - Indicate when a request is in progress.

- **Data State:**
  - Store and provide fetched data in a usable format.

- **Error Handling:**
  - Capture and expose errors that occur during the fetch process.

#### **c. Performance Optimizations**

- **Caching:**
  - Implement caching strategies to prevent redundant network requests.
  - Support cache invalidation mechanisms.

- **Abort Controllers:**
  - Use `AbortController` to cancel ongoing requests when the component unmounts or when a new request is initiated.

#### **d. Reusability and Flexibility**

- **Configurable Parameters:**
  - Allow users to customize aspects like refetch intervals, retry logic, etc.

- **Dependency Management:**
  - Enable refetching based on dependencies or specific triggers.

#### **e. Advanced Features**

- **Pagination Support:**
  - Handle paginated data fetching with ease.

- **Authentication:**
  - Support inclusion of authentication tokens or other security measures in requests.

- **Global State Integration:**
  - Integrate with global state management solutions (like Redux or Context API) if necessary.

#### **f. Type Safety and Documentation**

- **TypeScript Support:**
  - Provide strong typing for inputs and outputs to prevent common bugs.

- **Comprehensive Documentation:**
  - Clear documentation on usage, parameters, and return values.


### ** Solution **
```
import {
    useState,
    useEffect,
    useCallback,
    useRef
} from 'react';

interface FetchOptions {
    method ? : 'GET' | 'POST' | 'PUT' | 'DELETE';
    headers ? : HeadersInit;
    body ? : BodyInit | null;
    queryParams ? : Record < string, string | number | boolean > ;
    authToken ? : string
}

interface FetchState < T > {
    data: T | null;
    loading: boolean;
    error: string | null;
}

interface UseFetchReturn < T > extends FetchState < T > {
    refetch: () => void;
}

const useFetch = < T > (url: string, options: FetchOptions = {}): UseFetchReturn < T > => {
    const [data, setData] = useState < T | null > (null);
    const [loading, setLoading] = useState < boolean > (false);
    const [error, setError] = useState < string | null > (null);
    const cache = useRef < {
        [key: string]: T
    } > ({});
    const controllerRef = useRef < AbortController | null > (null);

    const buildUrlWithParams = useCallback((url: string, queryParams ? : Record < string, string | number | boolean > ) => {
        if (!queryParams) return url;
        const query = new URLSearchParams();
        Object.entries(queryParams).forEach(([key, value]) => {
            query.append(key, String(value));
        });
        return `${url}?${query.toString()}`;
    }, []);

    const fetchData = useCallback(async () => {
        const fetchUrl = await buildUrlWithParams(url, options.queryParams);
        if (cache.current[fetchUrl]) {
            setData(cache.current[fetchUrl]);
            return;
        }

        setLoading(true);
        setError(null);

        controllerRef.current = new AbortController();

        const prepareHeaders: HeadersInit = {
            'Content-Type': 'application/json',
            ...(options.authToken ? {
                Authorization: `Bearer ${options.authToken}`
            } : {}),
            ...options.headers,
        };

        let prepareBody: BodyInit | null = options.body;
        if (
            headers['Content-Type'] === 'application/json' &&
            typeof options.body === 'object' &&
            options.body !== null
        ) {
            preparedBody = JSON.stringify(options.body);
        }

        try {
            const response = await fetch(fetchUrl, {
                method: options.method || 'GET',
                headers: prepareHeaders,
                body: prepareBody,
                signal: controllerRef.current.signal,
            });

            if (!response.ok) {
                throw new Error(`Error: ${response.status} - ${response.statusText}`);
            }

            const result: T = await response.json();
            setData(result);
            if (options.cacheKey) {
                cache.current[options.cacheKey] = result;
            }
        } catch (err: any) {
            if (err.name !== 'AbortError') {
                setError(err.message);
            }
        } finally {
            setLoading(false);
        }
    }, [url, options, buildUrlWithParams]);

    useEffect(() => {
        fetchData();

        return () => {
            controllerRef.current?.abort();
        };
    }, [fetchData]);

    const refetch = useCallback(() => {
        controllerRef.current?.abort();
        fetchData();
    }, [fetchData]);

    return {
        data,
        loading,
        error,
        refetch
    };
};

export default useFetch;

```
---









## **`useDebounce` Custom Hook**

### **Functionalities and Expectations**

A well-designed `useDebounce` hook should provide a simple yet flexible way to debounce rapidly changing values, such as user inputs. Here are the key functionalities and areas to evaluate:

#### **a. Core Debouncing Logic**

- **Delay Configuration:**
  - Allow customization of the debounce delay time.

- **Value Tracking:**
  - Return a debounced value that updates only after the specified delay.

#### **b. Flexibility and Reusability**

- **Immediate Execution Option:**
  - Optionally execute a function immediately before debouncing.

- **Cancelation Mechanism:**
  - Provide a way to cancel the debounce effect if needed.

#### **c. Cleanup and Performance**

- **Effect Cleanup:**
  - Ensure that timers are properly cleared to prevent memory leaks.

- **Optimized Performance:**
  - Minimize unnecessary re-renders by using appropriate dependencies.

#### **d. Type Safety and Documentation**

- **TypeScript Support:**
  - Ensure type safety for input values and debounced outputs.

- **Comprehensive Documentation:**
  - Clear instructions on usage, parameters, and return values.

### ** Solution **

```
import {
    useState,
    useEffect,
    useRef,
    useCallback
} from 'react';

interface UseDebounceOptions {
    delay ? : number;
    immediate ? : boolean;
}

function useDebounce < T > (
    value: T,
    options: UseDebounceOptions = {}
): {
    debouncedValue: T;
    cancel: () => void;
    flush: () => void;
} {
    const { delay = 300, immediate = false } = options;
    const [debouncedValue, setDebouncedValue] = useState < T > (value);
    const timerRef = useRef < NodeJS.Timeout | null > (null);
    const immediateRef = useRef < boolean > (immediate);
    const latestValueRef = useRef < T > (value);

    const cancel = useCallback(() => {
        if (timerRef.current) {
            clearTimeout(timerRef.current);
            timerRef.current = null;
        }
    }, []);

    const flush = useCallback(() => {
        if (timerRef.current) {
            clearTimeout(timerRef.current);
            setDebouncedValue(latestValueRef.current);
            timerRef.current = null;
        }
    }, []);

    useEffect(() => {
        latestValueRef.current = value;

        if (immediateRef.current) {
            setDebouncedValue(value);
            immediateRef.current = false;
            return;
        }

        cancel();

        timerRef.current = setTimeout(() => {
            setDebouncedValue(value);
        }, delay);

        return () => {
            cancel();
        };
    }, [value, delay, cancel]);

    return {
        debouncedValue,
        cancel,
        flush
    };
}

export default useDebounce;

```






---
