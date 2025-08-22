# Advanced USSD Builder

<div align="center">

![npm version](https://img.shields.io/npm/v/advanced-ussd-builder.svg?style=flat-square)
![npm downloads](https://img.shields.io/npm/dm/advanced-ussd-builder.svg?style=flat-square)
![license](https://img.shields.io/npm/l/advanced-ussd-builder.svg?style=flat-square)
![build status](https://img.shields.io/github/actions/workflow/status/iammohammedb/advanced-ussd-builder/ci.yml?branch=master&style=flat-square)
![typescript](https://img.shields.io/badge/TypeScript-4.5+-blue.svg?style=flat-square)

**A powerful TypeScript library for building interactive USSD menus with persistent state management**

[Installation](#installation) • [Quick Start](#quick-start) • [Documentation](#core-concepts) • [Examples](#usage-examples) • [API](#api-reference)

</div>

## 📋 Table of Contents

- [Features](#features)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Provider Support](#provider-support)
- [Core Concepts](#core-concepts)
  - [UssdMenu](#ussdmenu)
  - [UssdBuilder](#ussdbuilder)
  - [Session Management](#session-management)
- [Usage Examples](#usage-examples)
  - [Render Menu](#render-menu-display-options)
  - [Render Menu with Function Handler](#render-menu-with-function-handler)
  - [Basic Menu](#basic-menu)
  - [Dynamic Menus](#dynamic-menus)
  - [Proxy Menu](#proxy-menu-external-service-integration)
  - [Handler Menus](#handler-menus)
  - [Conditional Menu Rendering](#conditional-menu-rendering)
- [API Reference](#api-reference)
- [Provider Integration](#provider-integration)
  - [MTN](#mtn)
  - [Nalo](#nalo)
  - [Telecel](#telecel)
  - [Airtel-Tigo](#airtel-tigo)
  - [Cross-Switch](#cross-switch)
  - [Custom Provider](#custom-provider)
- [Advanced Features](#advanced-features)
  - [Session State Management](#session-state-management)
  - [Middleware Support](#middleware-support)
  - [Authentication Handler](#authentication-handler)
  - [Proxy Service Integration](#proxy-service-integration)
  - [Path-based Handler Modules](#path-based-handler-modules)
- [License](#license)

## ✨ Features

- 🚀 **Multi-Provider Support** - Works with MTN, Nalo, Telecel, Airtel-Tigo, Cross-Switch, and Custom providers
- 🔐 **Built-in Authentication** - PIN-based authentication with customizable handlers
- 🔌 **Middleware Support** - Request/response interceptors for logging, validation, and transformation
- 🎨 **Custom Provider Setup** - Easy integration with any USSD gateway through flexible configuration
- 🔀 **Proxy Support** - Reverse proxy capability to integrate external USSD services seamlessly
- 💾 **Persistent State Management** - Redis-powered session management for stateful interactions
- 📱 **Dynamic Menu Generation** - Create menus programmatically with render and handler functions
- 🔄 **Navigation History** - Track and navigate through menu paths seamlessly
- 🎯 **TypeScript Support** - Full type definitions for better development experience
- ⚡ **High Performance** - Optimized for speed with minified production builds
- 🔧 **Flexible Configuration** - Customizable session prefixes, navigation keys, and more

## 📦 Installation

```bash
npm install advanced-ussd-builder
```

or using yarn:

```bash
yarn add advanced-ussd-builder
```

### Prerequisites

- Node.js >= 14.x
- Redis server (for session management)
- TypeScript (for development)

## 🚀 Quick Start

```typescript
import { UssdBuilder, UssdMenu, iUssdMenu, iUssdMenuOptions, iMTNUSSDArgs } from 'advanced-ussd-builder';

// Define menu configuration
const menuOptions: iUssdMenuOptions = {
  provider: 'mtn',
  require_pin: false,
  service_code: '*123#',
  next_page_key: '#',
  back_page_key: '0',
  redis_connection_url: 'redis://localhost:6379',
  state: {
    user_id: '',
    session_data: {}
  }
};

// Create handler menus with path-based handlers
const registrationMenu: iUssdMenu = {
  title: 'Register',
  type: 'handler',
  handler_type: 'path',
  handler_name_in_destination: 'root_menu',
  handler: path.join(__dirname, '/handlers/registration.js')
};

// Create handler menus with function handlers
const balanceMenu: iUssdMenu = {
  title: 'Check Balance',
  type: 'handler',
  handler_type: 'function',
  handler: async (args) => {
    const balance = await getUserBalance(args.msisdn);
    return `Your balance is $${balance}`;
  }
};

// Create menu array
const menus = [registrationMenu, balanceMenu];

// Initialize USSD Builder with root menu
const ussdBuilder = new UssdBuilder(
  menuOptions,
  new UssdMenu('Welcome to MyApp', menus)
);

// Process USSD request (Express.js example)
app.post('/ussd/mtn', async (req, res) => {
  const ussdArgs = req.body as iMTNUSSDArgs;
  
  const sendResponse = (response) => {
    return res.json(response);
  };
  
  await ussdBuilder.pipe<'mtn'>(ussdArgs, sendResponse);
});
```

## 🌐 Provider Support

Advanced USSD Builder supports multiple telecom providers out of the box:

| Provider | Status | Request Interface | Response Interface |
|----------|--------|------------------|-------------------|
| MTN | ✅ Supported | `iMTNUSSDArgs` | `iMTNUssdCallback` |
| Nalo | ✅ Supported | `iNaloUSSDArgs` | `iNaloUssdCallback` |
| Telecel | ✅ Supported | `iTelecelUssdArgs` | `iTelecelUssdCallBack` |
| Airtel-Tigo | ✅ Supported | `iAirtelTigoUssdArgs` | `iAirtelTigoUssdCallBack` |
| Cross-Switch | ✅ Supported | `iCrossSwitchUssdArgs` | `iCrossSwitchUssdCallBack` |
| Custom | ✅ Supported | Custom Interface | Custom Response |

## 📚 Core Concepts

### UssdMenu

The `UssdMenu` class represents a menu container that can hold multiple menu items. Each menu item follows the `iUssdMenu` interface structure.

```typescript
// Create a root menu with multiple options
const menuItems: iUssdMenu[] = [
  {
    title: 'Option 1',
    type: 'handler',
    handler_type: 'function',
    handler: async (args) => {
      return 'Response for option 1';
    }
  },
  {
    title: 'Option 2',
    type: 'handler',
    handler_type: 'path',
    handler_name_in_destination: 'default',
    handler: './handlers/option2.js'
  }
];

const rootMenu = new UssdMenu('Select an option:', menuItems);
```

### UssdBuilder

The main orchestrator that manages menus, sessions, and request processing. It requires both configuration options and a root menu.

```typescript
const options: iUssdMenuOptions = {
  provider: 'mtn', // or 'nalo', 'telecel', 'airtel-tigo', 'cross-switch', 'custom'
  require_pin: false,
  service_code: '*123#',
  next_page_key: '#',
  back_page_key: '0',
  redis_connection_url: 'redis://localhost:6379',
  state: {}, // Custom state object
  middleware: async (args) => {
    // Optional middleware
  },
  authentication_handler: async (args) => {
    // Optional authentication
    return true;
  }
};

const builder = new UssdBuilder(options, rootMenu);
```

### Session Management

Sessions are automatically managed using Redis with the following structure:

- **Session Key**: `{prefix}:{session_id}`
- **Session Data**: Stores navigation path, current menu, and custom data
- **TTL**: Configurable time-to-live for session expiration

## 💡 Usage Examples

### Render Menu (Display Options)

```typescript
// Render menu displays a list of options to the user
const renderMenus: iUssdMenu[] = [
  {
    title: 'Account Services',
    type: 'render', // This will display as an option
    render: {
      success_message: 'Select an option:',
      error_message: 'Invalid selection'
    }
  },
  {
    title: 'Transfer Money',
    type: 'render',
    render: {
      success_message: 'Choose transfer type:'
    }
  },
  {
    title: 'Buy Airtime',
    type: 'render',
    render: {
      success_message: 'Select amount:'
    }
  }
];

const ussdBuilder = new UssdBuilder(
  options,
  new UssdMenu('Main Menu\n1. Account Services\n2. Transfer Money\n3. Buy Airtime', renderMenus)
);
```

### Render Menu with Function Handler

```typescript
// When type is 'render' with a handler, the handler controls the response
const dynamicRenderMenu: iUssdMenu = {
  title: 'Account List',
  type: 'render',
  handler_type: 'function',
  handler: async (args) => {
    // Dynamically generate menu options
    const accounts = await getUserAccounts(args.msisdn);
    const menuOptions = accounts.map((acc, idx) => 
      `${idx + 1}. ${acc.name} - ${acc.balance}`
    ).join('\n');
    
    // Handler returns the complete response
    return {
      message: `Your Accounts:\n${menuOptions}\n\nSelect account:`,
      continue_session: true
    };
  }
  // No render object needed when handler is provided
};
```

### Basic Menu

```typescript
const bankingMenus: iUssdMenu[] = [
  {
    title: 'Check Balance',
    type: 'handler',
    handler_type: 'function',
    handler: async (args) => {
      const balance = await getBalance(args.msisdn);
      return `Your balance: $${balance}`;
    }
  },
  {
    title: 'Transfer Funds',
    type: 'handler',
    handler_type: 'path',
    handler_name_in_destination: 'root_menu',
    handler: './handlers/transfer.js'
  },
  {
    title: 'Mini Statement',
    type: 'handler',
    handler_type: 'function',
    handler: async (args) => {
      const statement = await getStatement(args.msisdn);
      return statement;
    }
  }
];

const ussdBuilder = new UssdBuilder(
  options,
  new UssdMenu('Welcome to Banking Services', bankingMenus)
);
```

### Dynamic Menus

```typescript
// Dynamically create menus based on user data
const createDynamicMenu = async (userPhone: string) => {
  const user = await getUserData(userPhone);
  const menuItems: iUssdMenu[] = [];
  
  // Add different menus based on user status
  if (user.isRegistered) {
    menuItems.push({
      title: 'My Account',
      type: 'handler',
      handler_type: 'path',
      handler_name_in_destination: 'root_menu',
      handler: './handlers/account.js'
    });
  } else {
    menuItems.push({
      title: 'Register',
      type: 'handler',
      handler_type: 'path',
      handler_name_in_destination: 'root_menu',
      handler: './handlers/registration.js'
    });
  }
  
  return new UssdMenu('Welcome', menuItems);
};

// Use in request handler
app.post('/ussd', async (req, res) => {
  const rootMenu = await createDynamicMenu(req.body.msisdn);
  const builder = new UssdBuilder(options, rootMenu);
  await builder.pipe<'mtn'>(req.body, (response) => res.json(response));
});
```

### Proxy Menu (External Service Integration)

```typescript
// Proxy menus forward requests to external USSD services
const proxyMenus: iUssdMenu[] = [
  {
    title: 'Loan Services',
    type: 'proxy',
    proxy_config: {
      target_url: 'https://loan-service.example.com/ussd',
      timeout: 25000,
      headers: {
        'API-Key': process.env.LOAN_API_KEY,
        'Partner-ID': 'partner-123'
      }
    }
  },
  {
    title: 'Insurance Portal',
    type: 'proxy',
    proxy_config: {
      target_url: 'https://insurance.example.com/ussd',
      timeout: 20000,
      retry_attempts: 2,
      retry_delay: 1500,
      // Transform request/response for compatibility
      transform_request: (data) => ({
        phone: data.msisdn,
        session: data.session_id,
        input: data.user_input
      }),
      transform_response: (response) => ({
        message: response.text,
        require_feedback: response.continue
      })
    }
  }
];

// Combine local and proxy menus
const mainMenus: iUssdMenu[] = [
  {
    title: 'My Account',
    type: 'handler',
    handler_type: 'path',
    handler: './handlers/account.js'
  },
  ...proxyMenus // External services appear as regular menu items
];

const ussdBuilder = new UssdBuilder(
  options,
  new UssdMenu('Select Service:', mainMenus)
);
```

### Handler Menus

```typescript
// Simple handler
const balanceHandler = new UssdMenu('check_balance', async (args) => {
  const balance = await getBalance(args.msisdn);
  return `Your balance: $${balance}\n\nPress 0 to go back`;
});

// Handler with module path
const complexHandler = new UssdMenu('process_transfer', './handlers/transfer.js');

builder.register_menu(balanceHandler);
builder.register_menu(complexHandler);
```

### Paginated Menus

Pagination is automatic when menu items exceed the configured limit:

```typescript
const builder = new UssdBuilder({
  redis_url: 'redis://localhost:6379',
  max_menu_items: 5 // Show 5 items per page
});

const largeMenu = new UssdMenu('products', 'Select Product');
// Adding more than 5 items will trigger pagination
for (let i = 1; i <= 20; i++) {
  largeMenu.add_child(`product_${i}`, `Product ${i}`);
}
```

## 📖 API Reference

### UssdBuilder

#### Constructor Options

```typescript
interface UssdBuilderOptions {
  redis_url: string;           // Redis connection URL
  session_prefix?: string;     // Session key prefix (default: 'ussd')
  session_ttl?: number;        // Session TTL in seconds (default: 300)
  max_menu_items?: number;     // Max items per page (default: 5)
  end_session_text?: string;   // Custom end session message
}
```

#### Methods

| Method | Description | Parameters | Returns |
|--------|-------------|------------|---------|
| `pipe<T>(args, callback)` | Process USSD request | `args`: provider args, `callback`: response handler | Promise<void> |
| `getSession()` | Get current session | - | iUssdSession |
| `saveSession()` | Save session to Redis | - | Promise<void> |
| `endSession()` | End current session | - | Promise<void> |

### UssdMenu

#### Constructor

```typescript
new UssdMenu(
  id: string,
  title: string | Function | undefined,
  opts?: {
    custom_input?: boolean;
    module_path?: string;
  }
)
```

#### Menu Item Types

| Property | Description | Type |
|----------|-------------|------|
| `title` | Menu item display text | string |
| `type` | Menu type | 'render' \| 'handler' \| 'proxy' |
| `handler_type` | Handler type (for handler menus) | 'path' \| 'function' |
| `handler` | Handler implementation | string \| Function |
| `handler_name_in_destination` | Exported function name in module | string |
| `proxy_config` | Proxy configuration (for proxy menus) | ProxyConfig object |

## 🔌 Provider Integration

### MTN

```typescript
app.post('/ussd/mtn', async (req, res) => {
  const ussdArgs = req.body as iMTNUSSDArgs;
  
  // Response callback
  const sendResponse = <iMTNUssdCallback>(response: iMTNUssdCallback) => {
    return res.json(response);
  };
  
  // Configure and build menu
  const options: iUssdMenuOptions = {
    provider: 'mtn',
    service_code: '*123#',
    next_page_key: '#',
    back_page_key: '0',
    require_pin: false,
    redis_connection_url: REDIS_URL,
    state: { /* custom state */ }
  };
  
  const menus: iUssdMenu[] = [/* your menus */];
  const ussdBuilder = new UssdBuilder(
    options,
    new UssdMenu('Welcome', menus)
  );
  
  await ussdBuilder.pipe<'mtn'>(ussdArgs, sendResponse);
});
```

### Nalo

```typescript
app.post('/ussd/nalo', async (req, res) => {
  const ussdArgs = req.body as iNaloUSSDArgs;
  
  const sendResponse = <iNaloUssdCallback>(response: iNaloUssdCallback) => {
    return res.json(response);
  };
  
  const options: iUssdMenuOptions = {
    provider: 'nalo',
    service_code: '*123#',
    next_page_key: '#',
    back_page_key: '0',
    require_pin: false,
    redis_connection_url: REDIS_URL
  };
  
  const ussdBuilder = new UssdBuilder(options, rootMenu);
  await ussdBuilder.pipe<'nalo'>(ussdArgs, sendResponse);
});
```

### Telecel

```typescript
app.post('/ussd/telecel', async (req, res) => {
  const ussdArgs = req.body as iTelecelUssdArgs;
  
  const sendResponse = <iTelecelUssdCallBack>(response: iTelecelUssdCallBack) => {
    return res.json(response);
  };
  
  const options: iUssdMenuOptions = {
    provider: 'telecel',
    service_code: '*123#',
    next_page_key: '#',
    back_page_key: '0',
    require_pin: false,
    redis_connection_url: REDIS_URL
  };
  
  const ussdBuilder = new UssdBuilder(options, rootMenu);
  await ussdBuilder.pipe<'telecel'>(ussdArgs, sendResponse);
});
```

### Airtel-Tigo

```typescript
app.post('/ussd/at', async (req, res) => {
  const ussdArgs = req.body as iAirtelTigoUssdArgs;
  
  const sendResponse = <iAirtelTigoUssdCallBack>(response: iAirtelTigoUssdCallBack) => {
    return res.json(response);
  };
  
  const options: iUssdMenuOptions = {
    provider: 'airtel-tigo',
    service_code: '*123#',
    next_page_key: '#',
    back_page_key: '0',
    require_pin: false,
    redis_connection_url: REDIS_URL
  };
  
  const ussdBuilder = new UssdBuilder(options, rootMenu);
  await ussdBuilder.pipe<'airtel-tigo'>(ussdArgs, sendResponse);
});
```

### Cross-Switch

```typescript
app.post('/ussd/cross-switch', async (req, res) => {
  const ussdArgs = req.body as iCrossSwitchUssdArgs;
  
  const sendResponse = <iCrossSwitchUssdCallBack>(response: iCrossSwitchUssdCallBack) => {
    return res.json(response);
  };
  
  const options: iUssdMenuOptions = {
    provider: 'cross-switch',
    service_code: '*123#',
    next_page_key: '#',
    back_page_key: '0',
    require_pin: false,
    redis_connection_url: REDIS_URL
  };
  
  const ussdBuilder = new UssdBuilder(options, rootMenu);
  await ussdBuilder.pipe<'cross-switch'>(ussdArgs, sendResponse);
});
```

### Custom Provider

The library supports custom providers for any USSD gateway not listed above:

```typescript
// Custom provider implementation
app.post('/ussd/custom', async (req, res) => {
  const customArgs = {
    msisdn: req.body.phoneNumber,
    session_id: req.body.sessionId,
    msg: req.body.userInput,
    // Map your custom fields
  };
  
  const sendResponse = (response) => {
    // Transform response to your custom format
    const customResponse = {
      text: response.message || response.msg,
      continueSession: response.msgtype !== false,
      sessionId: response.session_id
    };
    return res.json(customResponse);
  };
  
  const options: iUssdMenuOptions = {
    provider: 'custom',
    service_code: req.body.serviceCode,
    next_page_key: '#',
    back_page_key: '0',
    require_pin: false,
    redis_connection_url: REDIS_URL,
    make_provider_response: false, // Handle response formatting manually
    state: {}
  };
  
  const menus: iUssdMenu[] = [/* your menus */];
  const ussdBuilder = new UssdBuilder(
    options,
    new UssdMenu('Welcome', menus)
  );
  
  await ussdBuilder.pipe<'custom'>(customArgs, sendResponse);
});
```

## 🚀 Advanced Features

### Session State Management

```typescript
// Pass state through options that persists across menus
const options: iUssdMenuOptions = {
  provider: 'mtn',
  service_code: '*123#',
  next_page_key: '#',
  back_page_key: '0',
  require_pin: false,
  redis_connection_url: REDIS_URL,
  state: {
    user_id: '12345',
    user_name: 'John Doe',
    account_type: 'premium',
    // Any custom data you need
  }
};

// Access state in handlers
const menuHandler: iUssdMenu = {
  title: 'Account Info',
  type: 'handler',
  handler_type: 'function',
  handler: async (args) => {
    // Access state passed through options
    const userId = args.state?.user_id;
    const userName = args.state?.user_name;
    return `Welcome ${userName}, your ID is ${userId}`;
  }
};
```

### Middleware Support

```typescript
const options: iUssdMenuOptions = {
  provider: 'mtn',
  service_code: '*123#',
  next_page_key: '#',
  back_page_key: '0',
  require_pin: false,
  redis_connection_url: REDIS_URL,
  middleware: async (args) => {
    // Log every request
    console.log('USSD Request:', args);
    
    // Validate user
    const user = await validateUser(args.msisdn);
    if (!user) {
      throw new Error('Unauthorized');
    }
    
    // Add user to args
    args.user = user;
  }
};
```

### Authentication Handler

```typescript
const options: iUssdMenuOptions = {
  provider: 'mtn',
  service_code: '*123#',
  next_page_key: '#',
  back_page_key: '0',
  require_pin: true, // Enable PIN requirement
  redis_connection_url: REDIS_URL,
  authentication_handler: async (args) => {
    // Custom authentication logic
    const isValid = await validateUserPin(args.msisdn, args.input);
    return isValid;
  }
};
```

### Proxy Service Integration

```typescript
// Advanced proxy configuration with all options
const advancedProxyMenu: iUssdMenu = {
  title: 'Payment Gateway',
  type: 'proxy',
  proxy_config: {
    target_url: 'https://payment.example.com/ussd',
    timeout: 25000,
    retry_attempts: 2,
    retry_delay: 1000,
    session_bridge: true, // Share session state
    headers: {
      'Authorization': `Bearer ${process.env.AUTH_TOKEN}`,
      'X-Partner-ID': 'partner-123'
    },
    // Transform outgoing request
    transform_request: (data) => ({
      phoneNumber: data.msisdn,
      sessionId: data.session_id,
      userInput: data.user_input,
      metadata: {
        provider: data.provider,
        timestamp: Date.now()
      }
    }),
    // Transform incoming response
    transform_response: (response) => ({
      message: response.display_text,
      require_feedback: response.await_input === true,
      session_data: response.session_state
    })
  }
};

// Use Cases for Proxy:
// - Integrate third-party services (loans, insurance, payments)
// - Implement microservices architecture
// - Connect partner USSD applications
// - Load balance across multiple backends
```

### Path-based Handler Modules

```typescript
// menu-handler.js - External handler module
export const root_menu = async (args) => {
  // Handler logic
  const userData = await fetchUserData(args.msisdn);
  return `Welcome ${userData.name}`;
};

export const process_payment = async (args) => {
  // Another handler in same module
  return 'Payment processed';
};

// Usage in menu definition
const menus: iUssdMenu[] = [
  {
    title: 'User Info',
    type: 'handler',
    handler_type: 'path',
    handler_name_in_destination: 'root_menu', // Function to call
    handler: path.join(__dirname, '/handlers/menu-handler.js')
  },
  {
    title: 'Process Payment',
    type: 'handler',
    handler_type: 'path',
    handler_name_in_destination: 'process_payment', // Different function
    handler: path.join(__dirname, '/handlers/menu-handler.js')
  }
];
```

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 👤 Author

### Mohammed Bashiru

- GitHub: [@iammohammedb](https://github.com/MohammedBashiru/advanced-ussd-builder)

## 🙏 Acknowledgments

- Redis for reliable session management
- TypeScript for type safety
- Jest for testing framework
- All contributors and users of this library

## 📞 Support

- 🌐 Website: [www.bashtech.solutions](https://www.bashtech.solutions)
- 📧 Email: <info@bashtech.solutions>
- 🐛 Issues: [GitHub Issues](https://github.com/MohammedBashiru/advanced-ussd-builder/issues)
- 💬 Discussions: [GitHub Discussions](https://github.com/MohammedBashiru/advanced-ussd-builder/discussions)

---

Made with ❤️ by the BashTech Solutions Team
