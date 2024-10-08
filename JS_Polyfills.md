```
Function.prototype._call = function (ctx, ...args) {
    // Ensure that 'this' is callable
    if (typeof this !== 'function') {
        throw new TypeError(`${this} is not callable`);
    }

    // If ctx is null or undefined, default to globalThis
    // In strict mode, it remains as is
    const context = ctx !== null && ctx !== undefined ? Object(ctx) : globalThis;

    // Create a unique symbol to avoid property name collisions
    const fn = Symbol('fn');

    try {
        // Assign the function to the context using the unique symbol
        context[fn] = this;

        // Invoke the function with the provided arguments
        return context[fn](...args);
    } finally {
        // Clean up by deleting the temporary property
        delete context[fn];
    }
};
```
