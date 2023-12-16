# A Handy Checklist of Secure Coding Practices: Protect Your App and Your Users

## 1. Preventing SQL Injection
SQL injection is a common way for malicious actors to attack web applications by taking advantage of poorly sanitized user inputs to manipulate the database. This vulnerability can lead to unauthorized access to sensitive information, alteration or deletion of data, and even administrative control of the application. It is crucial to address this issue to maintain data integrity, build user trust, and prevent serious security breaches. Ignoring this vulnerability could result in data leaks, unauthorized access, and harm to your application's reputation.

**Bad example**:
```javascript
const userId = req.params.id;
const query = `SELECT * FROM users WHERE id = ${userId}`;
db.query(query, (err, result) => {
    // process the query result
})
```

**Good example**:
```javascript
const userId = req.params.id;
const query = 'SELECT * FROM users WHERE id = ?';
db.query(query, [userId], (err, result) => {
    // process the query result
})
```

## 2. Cross-Site Scripting (XSS)
These attacks can let hackers inject malicious scripts into web apps, which then get executed in the browsers of unsuspecting users. This can lead to all sorts of bad things like data theft, financial loss, and unauthorized access to your accounts. If you don't take steps to secure your frontend, you might end up losing the trust of your users and even facing legal consequences.

**Bad example**:
```javascript
import React, { useState } from 'react';

const Component = () => {
    const [inputText, setInputText] = useState('');
    const [htmlString, setHtmlString] = useState('');

    const handleInputChange = (e) => {
        setInputText(e.target.value);
    }

    const handleHtmlStringSubmit = () => {
        // inputText is used directly
        setHtmlString(`<div>${inputText}</div>`)
    }

    return (
        <div>
            <input type='text' value={inputText} onChange={handleInputChange} />
            <button onClick={handleHtmlStringSubmit}>Submit</button>
            <div>
                <h3>HTML:</h3>
                <div dangerouslySetInnerHTML={{ __html: htmlString }} />
            </div>
        </div>
    )
}

export default Component;
```

`Good example`:
```javascript
import React, { useState } from 'react';

const Component = () => {
    const [inputText, setInputText] = useState('')
    const [htmlString, setHtmlString] = useState('')

    const handleInputChange = (e) => {
		setInputText(e.target.value)
	}

    const handleHtmlStringSubmit = () => {
		setHtmlString(`<div>${inputText}</div>`)
	}

    return (
        <div>
            <input type='text' value={inputText} onChange={handleInputChange} />
            <button onClick={handleHtmlStringSubmit}>Submit</button>
            <div>
                <h3>HTML:</h3>
                {/* htmlString is sanitized properly */}
                <div dangerouslySetInnerHTML={{__html: DOMPurify.sanitize(htmlString)}} />
            </div>
        </div>
    )
}
export default Component;
```

## 3. Testing Security with Selenium
- Security testing is a crucial aspect of software development that allows us to identify and address potential vulnerabilities and weaknesses in our applications.
- It involves simulating real-world attack scenarios to assess how well our defendses hold up against malicious intent.

Example with **Security Testing** taken into consideration:
```java
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;

public class LoginTestGood {
    public static void main(String[] args) {
        WebDriver driver = new ChromeDriver();
        try {
            // step 1: open the login page
            driver.get("https://example.com/login");

            // step 2: Perform security test for SQL injection
            String maliciousInput = "'; DROP TABLE users; --"; // Simulating SQL injection payload
            WebElement usernameField = driver.findElement(By.name("username"));
            WebElement passwordField = driver.findElement(By.name("password"));
            usernameField.sendKeys(maliciousInput);
            passwordField.sendKeys("secret123");
            passwordField.submit();

            // step3: wait for the login to complete (here we wait for an element on the login page to check for failure)
            WebDriverWait wait = new WebDriverWait(driver, 10);
            wait.until(ExpectedConditions.presenceOfElementLocatedBy(By.cssSelector(".login-form")));

            System.out.println("Test (good) - SQL Injection attempt detected");
        } catch (Exception e) {
            System.err.println("Test (good) - Security test passed: " + e.getMessage());
        } finally {
            driver.quit()
        }
    }
}
```

## 4. Secure Password Storage and Handling
**Bad example**: is where we are storing passwords directly in DB, and this can compromise the security of our app.
```javascript
app.post('/register', (req, res) => {
    const username = req.body.username;
    const password = req.body.password;

    // store the user in the db with the password as-is
})
```

**Good example**: is where the password is encrypted and stored in DB.
```javascript
const bcrypt = require('bcrypt');

app.post('/register', async (req, res) => { 
    const username = req.body.username
	const password = req.body.password
 
	const saltRounds = 10
	const hashedPassword = await bcrypt.hash(password, saltRounds)

    // Store the user in the database with the hashedPassword
}
```

## 5. Cross-Site Request Forgery (CSRF) Protection
CSRF is an attack that tricks users into submitting unintended requests by exploiting their authenticated session.

Proper CSRF protection is essential to prevent attackers from performing unauthorized actions on behalf of the user.

Without CSRF protection, attackers can execute malicious actions on behalf of authenticated users, leading to account compromise and unauthorized data manipulation.

**Bad example**: where a form is created for changing passwords without the support of a CSRF token.
```javascript
import { useState } from 'react';

const PasswordChangeForm = () => {
    const [newPassword, setNewPassword] = useState('')

    const handleChange = (event) => {
        setNewPassword(event.target.value)
    }

    const handleSubmit = (event) => {
        event.preventDefault();

        // Send the password change request without CSRF token
        fetch('/change-password', {
            method: 'POST',
            body: JSON.stringify({ newPassword }),
            headers: {
                'Content-Type': 'application/json',
            },
        })
        .then((response) => response.json())
        .then((data) => {
            console.log(data)
			// Handle response data
        })
        .catch((e) => {
            console.error('Error:', error)
            // Handle error
        })
    }
}

export default PasswordChangeForm;
``` 

**Good example**: where the support of CSRF token is provided for better security.
```javascript
// ...

fetch('/change-password', {
    // ...
    headers: {
        'Content-Type': 'application/json',
        'CSRF-Token': csrfToken,
    }
    // ...
})
```

```javascript
export default PasswordChangeForm

const express = require('express')
const bodyParser = require('body-parser')
const csurf = require('csurf')

const app = express()
app.use(bodyParser.json())

// Generate and include CSRF token for all responses
app.use(csurf())

// Middleware to expose CSRF token to the frontend
app.use((req, res, next) => {
    res.cookie('XSRF-TOKEN', req.csrfToken())
    next()
})

app.post('/change-password', (req, res) => {
    // validate CSRF token before processing the request
    if(req.headers['csrf-token'] !== req.csrfToken()) {
        return res.status(403).json({ error: 'Invalid CSRF token.' })
    }

    // processing password change logic ...
})

// ...
```

## 6. Secure Session Management
- Secure Session Management is essential to prevent unauthorized access and session hijacking.
- If session management is insecure, it can lead to session hijacking, which allows attackers to impersonate users and gain access to sensitive information or perform actions on their behalf.

**Bad example**: where no session management is done
```javascript
// Insecure session management using just plain HTTP cookies
app.get('/dashboard', (req, res) => {
    const user = findUserById(req.cookies.userId);
    // display user dashboard logic...
})
```

**Good example**: where Redis is used for session management, making the system more secure.
```javascript
// secure session management
const session = require('express-session');
const RedisStore = require('connect-redis')(session);

const sessionConfig = {
    store: new RedisStore({ url: 'redis://localhost:6379' }),
    secret: 'secret_key',
    resave: false,
    saveUninitialized: false,
    cookie: {
        secure: true,
        httpOnly: true,
        maxAge: 3600000,
    }
}

app.use(session(sessionConfig));

app.get('/dashboard', (req, res) => {
    const user = findUserById(req.session.userId)
    // display user dashboard logic...
})
```

## 7. Content Security Policy (CSP) Implementation
- The Content Security Policy (CSP) plays a vital role in web application security by preventing Cross-Site Scripting (XSS) attacks. It does this by regulating the resources that are allowed to be loaded and executed on a web page.
- CSP accomplishes this by setting limits on which sources can be used for scripts, styles, fonts, images, and other resources, thereby reducing the potential risks of XSS vulnerabilities.

**Bad example**: where the React app does not include any CSP, leaving it vulnerable to various types of attacks, including Cross-Site Scripting (XSS).
```javascript
const ExampleApp = () => {
    return (
        <div>
            <h1>Welcome to My App</h1>
            <p>This is a vulnerable React application.</p>
            <script src='https://evil.com/malicious.js'></script>
        </div>
    )
}
export default ExampleApp
```

Good example: where the server includes a CSP middleware that sets the "Content-Security-Policy" header in the HTTP response. The CSP is configured to only allow script sources from the same origin ('self'), the trusted CDN domain ("http://trusted-cdn.com"), and scripts with the matching nonce value ('nonce-randomly_generated_nonce').
```javascript
const ExampleApp = () => {
    return (
        <div>
            // ...
            <script src='https://trusted-cdn.com/app.js' nonce='randomly_generated_nonce'></script>
        </div>
    )
}
export default ExampleApp;

const express = require('express');
const app = express();

// CSP middleware to set the Content-Security-Policy header
app.use((req, res, next) => {
    res.setHeader(
        'Content-Securtiy-Policy',
        "script-src 'self' 'nonce-randomly_generated_nonce' https://trusted-cdn.com; default-src 'self'"
    )
    next();
})

// Serve the React application
app.use(express.static('build'))
app.listen(3000, () => {
    // ...
})
```

## 8. Handling Error Messages Securely
When an application encounters issues, error messages can offer useful feedback to both users and developers. However, if these messages are not handled properly, they can reveal sensitive information or help attackers identify potential vulnerabilities.

**Bad example**: where the whole user object is sent in response.
```javascript
app.get('/user/:id', (req, res) => {
    const userId = req.params.id;
    const user = getUserById(userId)
    if(!user) {
        return res.status(404).json({ error: 'User not found' })
    }
    res.json(user);
})
```

**Good example**: where only necessary details are sent in exposed.
```javascript
app.get('/user/:id', (req, res) => {
    // ...
    res.json({ username: user.username, email: user.email })
})
```

## 9. Secure File Uploads
Users can use file upload functionality to share data with the application. However, if this feature is not handled securely, it can become a vector for various attacks, such as code execution or denial of service.

Insecure file uploads can have serious consequences, such as remote code execution or the storage of malicious files on the server.

Therefore, it is crucial to implement secure file upload practices to prevent potential threats and maintain the integrity of the application.

**Bad example**: where the application allows users to upload files without proper validation and handling. The uploaded files are stored in the "uploads" directory, making them publicly accessible, which is a security risk. Additionally, the application does not check the file type or perform any security checks, leaving it vulnerable to malicious file uploads that may contain harmful scripts or malware.
```javascript
const express = require('express')
const app = express()
const multer = require('multer')

app.use(express.static('uploads')) // Exposing the uploads directory

// Insecure file upload handling
const storage = multer.diskStorage({
    destination: (req, file, cb) => {
        cb(null, 'uploads/') // Uploading files to the "uploads" directory
    },
    filename: (req, file, cb) => {
        cb(null, file.originalname) // Using the original filename
    }
})

const upload = multer({ storage })
app.post('/uploads', upload.single(file), (req, res) => {
    // File processing logic (e.g., saving to the database)
	// ...
    res.send('File uploaded successfully.')
})
```

**Good example**: where the application implements secure file upload practices, including proper validation, file type checking, and storing files in a protected directory.
```javascript
const express = require('express')
const app = express()
const multer = require('multer')
const path = require('path')
const crypto = require('crypto')

// Secure file upload handling
const storage = multer.diskStorage({
    destination: (req, file, cb) => {
        cb(null, 'uploads/')
    },
    filename: (req, file, cb) => {
        // Generate a unique filename to prevent overwriting 
        const uniqueFilename = crypto.randomBytes(16).toString('hex');
        const fileExtension = path.extname(file.originalname);
        cb(null, uniqueFilename + fileExtension)
    }
})

const fileFilter = (req, file, cb) => {
    // Allow only specific file types (e.g., images)
    if(file.mimetype === 'image/jpeg' || file.mimetype === 'image/png' || file.mimetype === 'image/gif') {
        cb(null, true)
    } else {
        cb(new Error('Invalid file type. Only images (jpeg, png, gif) are allowed.'))
    }
}

const upload = multer({ storage, fileFilter })

app.post('/upload', upload.single('file'), (req, res) => {
	// File processing logic (e.g., saving to the database)
	// ...
	res.send('File uploaded successfully.')
})
// ...
```

## 10. Logging and Auditing for Security Monitoring 
**Bad example**:
```javascript
// Inadequate logging and auditing mechanisms
app.post('/data', (req, res) => {
    const sensitiveData = req.body.data
    // Process data without logging relevant events
})
``` 

**Good example**: where there is appropriate logging of sensitive actions and events, facilitating security monitoring and incident response.
```javascript
const fs = require('fs');

// Logging sensitive actions and events 
app.post('/data', (req, res) => {
    const sensitiveData = req.body.data;
    // Process data and log relevant events
    fs.appendFile('security.log', `Data processed: ${sensitiveData}\n`, (err) => {
        if(err) {
            console.error('Error writing to log file:', err)
        }
    })
})
```

