## Accessing Elements  

[Testing Library Cheatsheet](https://testing-library.com/docs/react-testing-library/cheatsheet/)  

Examples from LoginForm.test.tsx
```javascript
import { render, screen } from '@testing-library/react';

test('<LoginForm /> error text showing when no email text is given', () => {
    render(<LoginForm isLoading={false} isLoggedIn={false} /* ... */ />);

    const emailInput = screen.getByLabelText('E-Mail Adresse');
    const passwordInput = screen.getByLabelText('Passwort');

    const submitButton = screen.getByRole('button', { name: submitButtonLabel });
    userEvent.click(submitButton);
    
    expect(screen.getByText(/Bitte geben Sie eine gültige E-Mail Adresse ein/i)).toBeInTheDocument();
});
```
----
```javascript
import { render, screen } from '@testing-library/react';

test('<LoginForm /> password visibility button icon switch by click', () => {
    render(<LoginForm isLoading={false} isLoggedIn={false} /* ... */ />);

    expect(screen.getByTestId('visibility-icon-on')).toBeInTheDocument();

    const visibilityIconButton = screen.getByRole('button', { name: 'toggle password visibility' });
    userEvent.click(visibilityIconButton);

    expect(screen.queryByTestId('visibility-icon-on')).not.toBeInTheDocument();
    expect(screen.getByTestId('visibility-icon-off')).toBeInTheDocument();
});
```
----
```javascript
expect(element).toHaveAttribute('href', 'https://www.test.com');
expect(element).toHaveClass('classNameForTheLink');
expect(element).toHaveStyle('color: #000');
expect(getByTestId('headline').className).toContain('headlineSmall');

expect(backButton).toBeDisabled();
expect(backButton).not.toBeVisible();
```

### Using within  

[dom-testing-library/api-within/](https://testing-library.com/docs/dom-testing-library/api-within/)

```javascript
import {render, screen, within} from '@testing-library/react'

render(<MyComponent />)
const messages = screen.getByTestId('container-id')
const helloMessage = within(messages).getByText('hello')
```

## Hooks
### Hooks testing Library
[react-hooks-testing-library Docs](https://react-hooks-testing-library.com)  
[react-hooks-testing-library on GitHub](https://github.com/testing-library/react-hooks-testing-library)  

```javascript
it('checks useRenderHTML condition', () => {
    const testHTML1 = '<div>Just some text in a div</div>';
    const testHTML2 = '<p>Just another text in a p tag</p>';

    const { result: result1 } = renderHook(() => useRenderHTML(testHTML1));
    const { result: result2 } = renderHook(() => useRenderHTML(testHTML2));
    
    expect(result1.current).not.toBe(result2.current);
});

```
----

## Mocking Data  

[Understanding Jest Mocks](https://medium.com/@rickhanlonii/understanding-jest-mocks-f0046c68e53c)

### Mock a function with jest.fn  

```javascript
test('<LoginForm /> text typing and button click functionality', () => {
    const startSSOActionMock = jest.fn();
    
    render(<LoginForm startSSOAction={startSSOActionMock} />);
    
    const emailText = 'abcd@abc.de';
    const passwordText = '12345678';
    const emailInput = screen.getByLabelText('E-Mail Adresse');
    const passwordInput = screen.getByLabelText('Passwort');
    
    userEvent.type(emailInput, emailText);
    userEvent.type(passwordInput, passwordText);
    
    expect(emailInput).toHaveValue(emailText);
    expect(passwordInput).toHaveValue(passwordText);
    
    const submitButton = screen.getByRole('button', { name: 'Anmelden' });
    userEvent.click(submitButton);
    
    expect(startSSOActionMock).toHaveBeenCalledTimes(1);
    expect(startSSOActionMock).toHaveBeenCalledWith(emailText, passwordText);
});
```  
----

### Mock a function with jest.spyOn  

```javascript
import * as htmlToJsxParser from '@/lib/utils/htmlToJsxParser';

jest.spyOn(React.default, 'useContext').mockImplementation(
    function fnc(): ConfigContextProps { return  configContextValue }
);
jest.spyOn(htmlToJsxParser, 'default').mockImplementation(
    (htmlString: string) => htmlString
);

```
----
```javascript
anyFunctions.ts 

export function getAnything() {
    return "origin value";
}
```

```javascript
myApp.ts

import getAnything from "./anyFunctions";

export function getSpecialValue(s) {
    return s + " " + getAnything();
}
```  

```javascript
myApp.test.ts

import * as myFunctions from "./anyFunctions";
import getSpecialValue from "./myApp";

test("some special mocking", () => {
    const mockFN = jest.spyOn(myFunctions, "getAnything");

    // override the implementation
    mockFN.mockImplementation(() => "some new value");
    expect(getSpecialValue("Cool, its")).toEqual("Cool, its some new value");

    // restore the original implementation
    mockFN.mockRestore();
    expect(getSpecialValue("Ok, its")).toEqual("Ok, its origin value");
});
```
----

### Mock a module with jest.mock  

```javascript
jest.mock('next/head', () => {
    return {
        __esModule: true,
        default: function NextHead({ children }: { children: Array<React.ReactElement> }) {
            return <>{children}</>;
        },
    };
});

jest.mock('next/image', () => {
    return {
        __esModule: true,
        default: function NextImage(props: {}) {
            return <img {...props} />;
        },
    };
});

jest.mock('@/components/ImgMediaCard', () => ({
    ImgMediaCard: function imgMediaCard({ ratio, headline }: { ratio: string; headline: string }) {
        return (
            <div>
                {headline} {ratio}
            </div>
        );
    },
}));
```   
----
```javascript
jest.mock('@/lib/utils/environment', () => {
    return {
        esModule: true,
        get isClient() {
            return true;
        },
    };
});

const mockIsIOSApp = jest.fn().mockReturnValue(false);
jest.mock('@/lib/os', () => {
    return {
        esModule: true,
        get isIOSApp() {
            return mockIsIOSApp();
        },
    };
});

// Later in any Test
mockIsIOSApp.mockReturnValue(true);
```  
----
```javascript
jest.mock('@/components/SocialMedia/SocialLinks', () => {
    return {
        SocialLinks: function SocialLinks() {
            return <div>SocialLinks</div>;
        },
    };
});

it('should render without social links if !social', () => {
	configContextValue.config.social = undefined;

	render(
		<ConfigContext.Provider value={configContextValue}>
			<FooterBottom />
		</ConfigContext.Provider>
	);

	expect(screen.queryByText('SocialLinks')).not.toBeInTheDocument();
});
```  
----
```javascript
jest.mock('@/components/Video/KalturaPlayer/KalturaPlayer', () => ({
    KalturaPlayerSticky({ entryId, articleId }: KalturaGlobalStickyPlayerProps) {
        return (
            <div data-testid="kaltura-player-sticky">
                entryId: {entryId}, articleId: {articleId}
            </div>
        );
    },
}));

describe('<GlobalStickyPlayer />', () => {
    test('with entryId and articleId', () => {
        const { getByTestId } = render(
            <GlobalStickyPlayerContext.Provider value={{ entryId: '0_pf7x5zfj', articleId: '229239638' }}>
                <GlobalStickyPlayer />
            </GlobalStickyPlayerContext.Provider>
        );
        expect(getByTestId('kaltura-player-sticky').textContent).toBe('entryId: 0_pf7x5zfj, articleId: 229239638');
    });

    test('without provider, without video', () => {
        const { queryByTestId } = render(<GlobalStickyPlayer />);
        expect(queryByTestId('kaltura-player-sticky')).not.toBeInTheDocument();
    });
});
```  
