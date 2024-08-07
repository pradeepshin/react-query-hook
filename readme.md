# React Query Hook

A simple Node.js module to add two numbers.

## Installation

```javascript
// apiConfig.ts

interface ApiEndpoint {
    url: string;
    headers?: Record<string, string>;
    method?: 'GET' | 'POST' | 'PUT' | 'DELETE';
}

interface ApiConfig {
    [key: string]: ApiEndpoint;
}

const apiConfig: ApiConfig = {
    getUser: {
        url: 'https://api.example.com/users',
        method: 'GET',
        headers: {
            'Authorization': 'Bearer token_here',
            'Content-Type': 'application/json'
        }
    },
    createUser: {
        url: 'https://api.example.com/users',
        method: 'POST',
        headers: {
            'Authorization': 'Bearer token_here',
            'Content-Type': 'application/json'
        }
    },
    updateUser: {
        url: 'https://api.example.com/users',
        method: 'PUT',
        headers: {
            'Authorization': 'Bearer token_here',
            'Content-Type': 'application/json'
        }
    },
    // More API endpoints can be defined here
};

export default apiConfig;

```

```javascript
// useFetchData.ts

import { useQuery, UseQueryOptions } from 'react-query';
import apiConfig from './apiConfig'; // adjust the import path as necessary

interface FetchDataParams {
    key: keyof typeof apiConfig;
    queryParams?: Record<string, any>;
    body?: any;
}

const fetchData = async ({ key, queryParams, body }: FetchDataParams) => {
    const { url, method = 'GET', headers } = apiConfig[key];
    const queryString = new URLSearchParams(queryParams).toString();
    const fullUrl = queryString ? `${url}?${queryString}` : url;

    const config: RequestInit = {
        method,
        headers,
        body: body ? JSON.stringify(body) : null,
    };

    try {
        const response = await fetch(fullUrl, config);
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        return await response.json();
    } catch (error) {
        throw error;
    }
};

const useFetchData = <T>(params: FetchDataParams, options?: UseQueryOptions<T>) => {
    return useQuery<T>([apiConfig[params.key].url, params], () => fetchData(params), options);
};

export default useFetchData;
```

```javascript
// useSubmitData.ts

import { useMutation } from 'react-query';
import apiConfig from './apiConfig'; // adjust the import path as necessary

const submitData = async ({ key, body }: { key: keyof typeof apiConfig; body: any }) => {
    const { url, method = 'POST', headers } = apiConfig[key];

    const config: RequestInit = {
        method,
        headers,
        body: JSON.stringify(body),
    };

    try {
        const response = await fetch(url, config);
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        return await response.json();
    } catch (error) {
        throw error;
    }
};

const useSubmitData = () => {
    return useMutation(submitData);
};

export default useSubmitData;
```

```javascript
// GET request

import React from 'react';
import useFetchData from './useFetchData'; // adjust the import path as necessary

const UsersComponent = () => {
    const { data, isLoading, error } = useFetchData({
        key: 'getUser',
        queryParams: { userId: '123' } // example of query params
    });

    if (isLoading) return <div>Loading...</div>;
    if (error) return <div>Error occurred: {error.message}</div>;

    return (
        <div>
            <h1>User Details</h1>
            <pre>{JSON.stringify(data, null, 2)}</pre>
        </div>
    );
};

export default UsersComponent;
```

```javascript

// POST Request 
import React, { useState } from 'react';
import useFetchData from './useFetchData'; // adjust the import path as necessary

const CreateUserComponent = () => {
    const [userData, setUserData] = useState(null);
    const { data, isLoading, error } = useFetchData({
        key: 'createUser',
        body: { name: 'John Doe', email: 'john@example.com' } // example of body data
    });

    if (isLoading) return <div>Submitting data...</div>;
    if (error) return <div>Error occurred: {error.message}</div>;
    if (data) {
        // Assuming the API response includes the created user data
        setUserData(data);
    }

    return (
        <div>
            <h1>User Created</h1>
            <pre>{JSON.stringify(userData, null, 2)}</pre>
        </div>
    );
};

export default CreateUserComponent;


```

```javascript
// useFetchData.test.ts

import { renderHook } from '@testing-library/react-hooks';
import useFetchData from './useFetchData';
import apiConfig from './apiConfig';

global.fetch = jest.fn();

const mockUserResponse = { id: '123', name: 'John Doe' };

beforeEach(() => {
  jest.clearAllMocks();
  (fetch as jest.Mock).mockResolvedValue({
    ok: true,
    json: () => Promise.resolve(mockUserResponse),
  });
});

it('fetches data successfully using GET', async () => {
  const { result, waitFor } = renderHook(() =>
    useFetchData({ key: 'getUser' })
  );

  await waitFor(() => expect(result.current.isSuccess).toBe(true));

  expect(result.current.data).toEqual(mockUserResponse);
  expect(fetch).toHaveBeenCalledWith(`${apiConfig.getUser.url}`, {
    "body": null,
    "headers": apiConfig.getUser.headers,
    "method": "GET"
  });
});

it('fetches data successfully using POST', async () => {
  const mockData = { name: 'Jane Doe' };
  const { result, waitFor } = renderHook(() =>
    useFetchData({ key: 'createUser', body: mockData })
  );

  await waitFor(() => expect(result.current.isSuccess).toBe(true));

  expect(result.current.data).toEqual(mockUserResponse);
  expect(fetch).toHaveBeenCalledWith(apiConfig.createUser.url, {
    "body": JSON.stringify(mockData),
    "headers": apiConfig.createUser.headers,
    "method": "POST"
  });
});

// useSubmitData.test.ts

// actionTypes.js
export const actionTypes = {
  SET_STEP: 'SET_STEP',
  SET_PAYMENT_DETAILS: 'SET_PAYMENT_DETAILS',
  SET_BILLING_DETAILS: 'SET_BILLING_DETAILS',
  SET_LOADING: 'SET_LOADING',
  SET_ERROR: 'SET_ERROR'
};

// paymentReducer.js

import { actionTypes } from './actionTypes';

export const initialState = {
  step: 1,
  paymentDetails: {
    cardNumber: '',
    expirationDate: '',
    cvv: '',
    cardHolderName: ''
  },
  billingDetails: {
    address: '',
    city: '',
    postalCode: '',
    country: ''
  },
  isLoading: false,
  error: null
};



export function paymentReducer(state, action) {
  switch (action.type) {
    case actionTypes.SET_STEP:
      return { ...state, step: action.payload };
    case actionTypes.SET_PAYMENT_DETAILS:
      return { ...state, paymentDetails: { ...state.paymentDetails, ...action.payload } };
    case actionTypes.SET_BILLING_DETAILS:
      return { ...state, billingDetails: { ...state.billingDetails, ...action.payload } };
    case actionTypes.SET_LOADING:
      return { ...state, isLoading: action.payload };
    case actionTypes.SET_ERROR:
      return { ...state, error: action.payload };
    default:
      throw new Error(`Unknown action: ${action.type}`);
  }
}

// PaymentProvider.js

import React, { useReducer } from 'react';
import { PaymentContext } from './PaymentContext';
import { paymentReducer, initialState } from './paymentReducer';

export function PaymentProvider({ children }) {
  const [state, dispatch] = useReducer(paymentReducer, initialState);

  return (
    <PaymentContext.Provider value={{ state, dispatch }}>
      {children}
    </PaymentContext.Provider>
  );
}

// PaymentContext.js

import { createContext, useContext } from 'react';

export const PaymentContext = createContext();

export function usePaymentContext() {
  const context = useContext(PaymentContext);
  if (!context) {
    throw new Error('usePaymentContext must be used within a PaymentProvider');
  }
  return context;
}

// ======
import React, { useEffect } from 'react';
import { usePaymentContext } from '../context/PaymentContext';
import { useApi } from '../hooks/useApi';

function Step1() {
  const { state, dispatch } = usePaymentContext();
  const { data, isLoading, error } = useApi('/api/paymentDetails', 'POST', state.paymentDetails, [state.paymentDetails]);

  useEffect(() => {
    if (data) {
      // Handle successful API response
      console.log('Payment details saved:', data);
    }
    if (error) {
      dispatch({ type: 'SET_ERROR', payload: error });
    }
  }, [data, error, dispatch]);

  const handleChange = (e) => {
    dispatch({
      type: 'SET_PAYMENT_DETAILS',
      payload: { [e.target.name]: e.target.value }
    });
  };

  return (
    <div>
      <h2>Step 1: Payment Details</h2>
      {state.error && <p style={{ color: 'red' }}>{state.error}</p>}
      <form>
        <label>
          Card Number:
          <input
            type="text"
            name="cardNumber"
            value={state.paymentDetails.cardNumber}
            onChange={handleChange}
          />
        </label>
        <label>
          Expiration Date:
          <input
            type="text"
            name="expirationDate"
            value={state.paymentDetails.expirationDate}
            onChange={handleChange}
          />
        </label>
        <label>
          CVV:
          <input
            type="text"
            name="cvv"
            value={state.paymentDetails.cvv}
            onChange={handleChange}
          />
        </label>
        <label>
          Card Holder Name:
          <input
            type="text"
            name="cardHolderName"
            value={state.paymentDetails.cardHolderName}
            onChange={handleChange}
          />
        </label>
      </form>
      {isLoading && <p>Loading...</p>}
    </div>
  );
}

export default Step1;

// paymentReducer.test.js
import { paymentReducer, initialState } from './paymentReducer';
import { actionTypes } from './actionTypes';

describe('paymentReducer', () => {
  it('should return the initial state', () => {
    expect(paymentReducer(undefined, {})).toEqual(initialState);
  });

  it('should handle SET_STEP', () => {
    const action = { type: actionTypes.SET_STEP, payload: 2 };
    const expectedState = { ...initialState, step: 2 };
    expect(paymentReducer(initialState, action)).toEqual(expectedState);
  });

  it('should handle SET_PAYMENT_DETAILS', () => {
    const action = {
      type: actionTypes.SET_PAYMENT_DETAILS,
      payload: { cardNumber: '1234 5678 9012 3456' }
    };
    const expectedState = {
      ...initialState,
      paymentDetails: { ...initialState.paymentDetails, cardNumber: '1234 5678 9012 3456' }
    };
    expect(paymentReducer(initialState, action)).toEqual(expectedState);
  });

  // Add more tests for other action types as needed
});

// src/context/PaymentProvider.test.js
import React from 'react';
import { render, screen, fireEvent } from '@testing-library/react';
import { PaymentProvider, usePaymentContext } from './PaymentProvider';

const TestComponent = () => {
  const { state, dispatch } = usePaymentContext();
  return (
    <div>
      <span data-testid="step">{state.step}</span>
      <button
        onClick={() => dispatch({ type: 'SET_STEP', payload: 2 })}
      >
        Set Step
      </button>
    </div>
  );
};

describe('PaymentProvider', () => {
  it('should provide initial state', () => {
    render(
      <PaymentProvider>
        <TestComponent />
      </PaymentProvider>
    );
    expect(screen.getByTestId('step').textContent).toBe('1');
  });

  it('should update state on dispatch', () => {
    render(
      <PaymentProvider>
        <TestComponent />
      </PaymentProvider>
    );
    fireEvent.click(screen.getByText('Set Step'));
    expect(screen.getByTestId('step').textContent).toBe('2');
  });
});
```