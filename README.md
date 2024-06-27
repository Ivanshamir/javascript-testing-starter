# Mastering JavaScript Unit Testing

- we can install ui also for that write a new script: `vitest --ui`
- for view coverage in vitest, we will add new script in package.json: `vitest run --coverage`
- matchers:
    - toBe: used to for check primitive values like numbers, strings and booleans
    - toEqual: for comparing objects
- for truthiness
    - toBeTruthy
    - toBeFalsy
    - toBeNull
    - toBeUndefined
    - toBeDefined
- for numbers
    - toBeGreaterThan
    - toBeGreaterThanOrEqualTo
    - toBeLessThan
    - toBeLessThanOrEqualTo
    - toBeCloseTo (for floating numbers)
- for strings
    - toMatch (we can compare string or can use regular expression)
- for objects
    - toMatchObject
    - toHaveProperty
- arrays
    - toContain
    - toHaveLength
- exceptions
    - toThrowError

- some examples:
    - `expect(Array.isArray(coupons).toBe(true))`

- **Boundary testing** A testing technique where er focus on the edges or boundaries of input values.
- Parameterizd test:
```
it.each([
    { age: 15, county: 'US', result: false },
    { age: 16, county: 'US', result: true },
    { age: 17, county: 'US', result: true },
    { age: 16, county: 'UK', result: false },
    { age: 17, county: 'UK', result: true },
    { age: 18, county: 'UK', result: true },
])('should return $result for $age, $county', ({ age, country, result }) => {
    expect(canDrive(age, country)).toBe(result);   
})
```
- using callback in expect: we will use callback in expect func so that if error generate from the main function vitest/jest can handle this and not break the test. `expect(() => stack.pop()).toThrow(/empty/i)`
- what is mock function: a function that imitates the behavior of a real function. Like functionA() id dependent on functionB(), but we want to test functionA() so we need to isolate it from functionB(). To control test we will mock functionB(). So in program we will simulate different results of functionB() so we can test functionA(). Example:
```
it('test case', () => {
    const greet = vi.fn() // in vitest, we need to use vi for create mock function. generally it returns undefined like any other function in javascript
    greet.mockReturnedValue('Helo') // it wll return Helo
    greet()

    // for promise we can use mockResolvedValue
    greet.mockResolvedValue('Hello');
    greet().then(result => console.log(result));

    // we can also mock the function implementation
    greet.mockImplementation(name => 'Hello '+ name);
    console.log(greet('there')); // Hello there

    // now we can write assertion
    expect(greet).toHaveBeenCalled();
    expect(greet).toHaveBeenCalledWith('there); // test the correct parameter has been passed!
    expect(greet).toHaveBeenCalledOnce(); // test the mock function has been called only once.
})
```
- To replace a real function with mock function, first we need to mock the module like `util.js` file before describe function starts:
```
vi.mock('../src/libs/currency'); // here we mock entire file and now we can use any method and mock them
```
now in the describe function:
```
describe('getPriceInCurrency', () => {
  it('should return price in target currency', () => {
    vi.mocked(getExchangeRate).mockReturnValue(1.5);

    const price = getPriceInCurrency(10, 'AUD');

    expect(price).toBe(15);
  });
});
```
so in getPriceInCurrency function we call getExchangeRate which is located in `../src/libs/currency` file, so we mock the entire currency file first then we will use the mocked function, so we wrapped the method in `vi.mocked(getExchangeRate)`.
- for partial mock of a module:
```
vi.mock('../src/libs/email', async (importOriginal) => {  // here importOriginal name doesn't matter, can use any name
  const originalModule = await importOriginal();
  return {
    ...originalModule, // we keep the other function as it is for original implementation and only mocks sendEmail method
    sendEmail: vi.fn()
  };
});
```
- what is spy: sometimes we need to monitor the behavior of functions during test execution. It collects information about function called and result. 
```
describe('login', () => {
  it('should email the one-time login code', async () => {
    const email = 'name@domain.com';
    const spy = vi.spyOn(security, 'generateCode');  // so security is the module name and generateCode is the function, spyOn is similar to mock and we can use the methods of spyOn.

    await login(email);

    const securityCode = spy.mock.results[0].value.toString(); // we will first console the results and thet's why we need to use mock, console.log(spy.mock.results[0])
    expect(sendEmail).toHaveBeenCalledWith(email, securityCode);
  });
});
```
- mock is so far global object, so we need to remove it after test. Clearing mocks methods:
    - mockClear(): which clears of information
    - mockReset(): similar to mockClear() but it also implements the empty function.
    - mockRestore(): similar to mockClear() but it doesn't create empty function, it actually resets to original implementation. It actually needs fo spy. 
we can use a shortcut for clear mocks in beforeEach or afterEach
```
beforeEach(() => {
    vi.clearAllMocks()
})
```
now instead of doing this we can set it globally. So in root we create `vitest.config.js`:
```
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    clearMocks: true
  }
});
```
And restart the test process.
- mock date time: `vi.setSystemTime('2021-02-01 07:59')`
- as prettier file is also saved in bin folder of node_modules we can use this via npx: `npx prettier . --write`
- for linting in husky, in root create a file `.lintstagedrc.json` and add:
```
{
  "*.+(js|ts)": [
    "prettier --write",
    "eslint"
  ]
}
```
and in .husky folder in pre-commit file:
```
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

npx lint-staged
```
- before push we want to test all code and for that `npx husky add .husky/pre-push "npx vitest run"`, here we dont't want to run test in watch mode, so we will use this