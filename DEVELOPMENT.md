# Development

## Sample widget consumer

```js
import React, { useEffect, useState } from 'react';

const url = "http://localhost:5500/dist/cjs/index.js";

const _requires = {
  react: require("react")
};

const createRequires = dependencies => name => {
  const _dependencies = dependencies || {};

  if (!(name in _dependencies)) {
    throw new Error(`Could not require '${name}'. '${name}' does not exist in dependencies.`);
  }

  return _dependencies[name];
};

const useRemoteComponent = (url, requires) => {
  const [state, setState] = useState({ isLoading: true, error: undefined, component: undefined });

  useEffect(() => {
    (async () => {
      try {
        const data = await fetch(url)
        const dataText = await data.text();

        // Require
        const exports = {};
        const module = { exports };
        const requiresResolved = createRequires(requires);
        (new Function("require", "module", "exports", dataText))(requiresResolved, module, exports);

        // Set component
        setState({ isLoading: false, error: undefined, component: module.exports });
      } catch (err) {
        setState({ isLoading: false, error: err, component: undefined });
      }
    })();
  }, [url, requires]);

  return state;
}

function App() {
  const remote = useRemoteComponent(url, _requires);
  const RemoteComponent = remote.component ?? (() => <></>);

  return (
    <RemoteComponent />
  );
}

export default App;
```
