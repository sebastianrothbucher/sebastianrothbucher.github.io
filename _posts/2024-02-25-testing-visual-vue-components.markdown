---
layout: post
title: "Testing Vue components - halfway easily"
date:   2024-02-25 12:00:00 +0100
categories: javascript vue components testing
---

When it comes to testing visual components, first adivice is: don't do it - rather test store/actions/reducers/validation methods (i.e. pure functions). But if you really want (or must), here's a method with VueJS I found the most helpful of several: using Vue test-utils' `querySelector`-like API to extract content (this assumes Vue3 with Vite).

There's several alternatives to testing visual components: 

- end2end tests - which are notoriously flaky and hard to keep stable (and not a unit test)
- rendering (deep) - in any shape or form not a unit test (with all problems coming with it)
- rendering (shallow) the component to HTML and just using regular expressions
- rendering (shallow) the component to HTML and using e.g. `unexpected-dom` (or JSDOM)
- rendering (shallow) the component and using plain selector methods for text or a small subset of HTML

I found the last one the most effective - as it's the minimal, most straightforward solution. And the `querySelector`-like syntax works for most actual scenarios. Plus we can still use `findComponent` to check props of children. 

## What's needed

- Vue - I'm using [Vite](https://vitejs.dev/) along with [Vitest](https://vitest.dev/) (could also use Jest et al - both frameworks are super-similar)
- [Vue test-utils](https://test-utils.vuejs.org/)

## The Basics

Vue test-utils allows rendering a component with given props to HTML. I do prefer *shallow* rendering, i.e. sub-components just appear as `<sub-component></sub-component>` and not rendered. This keeps the scope of tests small enough. Here's how that works - given the following rendered example: 

```html
<div>
  <table>
    <tbody>
      <tr>
        <td>
          Test-Sth
          <small>subtext</small>
        </td>
        <td>
          Sth else
        </td>
      </tr>
    </tbody>
  </table>
  <my-sub></my-sub>
</div>
```

Now w/ in any test method (`it('what', () => { /* do some testing */})`), one can use:

```javascript
const wrapper = mount(MyComp, { shallow: true, props: {myprop: 'myvalue'} });
expect(wrapper.find('table td:nth-child(1)').text()).toMatch(/Test-Sth.*subtext/);
expect(wrapper.find('table td:nth-child(1)').html()).toMatch(/Test-Sh.*<small.*subtext<\/small>/);    
```

So it's possible to drill down to what we really want to test with everything CSS has got (and that is a lot) - and then write specific tests. 

## Mocking stores et al

Components might require (or import) things, esp. stores (which are great) - and we can mock those out in the usual way (vitest and jest are pretty similar here, as well): 

```javascript
vi.mock('../../stores/my-store', () => {
  const useMyStore: any = () => useMyStore; // return oneself - also allow access to mock functions in later assertions
  useMyStore.loadSth = vi.fn();
  useMyStore.sth = [{id: 42, title: 'Test-Sth', /*...*/}]
  return { useMyStore };
});
```

Now, the component will get a mock instead of the real dependency for the scope of the test. As we're using JS' duckt-typing, we can even drill down into objects returned from constructor functions (like `defineStore` is). 

Assuming the component needs to call `loadSth` on mounted, we can now test whether the invocation happened: 

```javascript
expect((useMyStore as any).loadOwned).toHaveBeenCalled(); // (no params on this one, but you get the idea)
```

## Checking sub-components

We can also use `findComponent` (see [API Reference](https://test-utils.vuejs.org/api/)) et al to check whether sub-components get the right properties. Here's an example: 

```javascript
const mySub = wrapper.findComponent(MySub);
expect(mySub.props('foo')?.bar).toBe(42);
```

## So...

...even though it's only second preferennce, there is a quite pragmatic approach to testing single components in the Vue ecosystem. 

Hope you found this useful - as ever: let me know what you think!
