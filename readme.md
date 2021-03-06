### Scatter

The new home of the EOS Scatter extension.
This repository is under construction.




## Installation


#### Getting the Chrome Extension files


**From Repository** - 
* Clone repository
* `npm install` to get dependencies
* `npm run build` to compile a `build` folder.


**From Zip File** - 
* [Download the zip file](https://github.com/EOSEssentials/Scatter/raw/master/scatter.zip)
* Create a folder somewhere and extract the zip file there.


#### Installing it into Chrome
* Open up **Chrome** and type `chrome://extensions/` into the url bar
* Click the `Load unpacked extension...` button and point it at the folder you just created/built 
(_the folder should have a manifest.json inside of it_).
![1](https://github.com/nsjames/Scatter/raw/master/chrome_ext.jpg)
![2](https://github.com/nsjames/Scatter/raw/master/chrome_ext2.jpg)



## Interacting with Scatter

#### Important: [Click here for a full diagram of the interaction flow.](https://drive.google.com/file/d/1QVw9WozcKjOQXO6uQKQVqaKt3C3lUr2F/view?usp=sharing)

There are some caveats to using Scatter, you should get familiar with them from the flow diagram before
continuing, otherwise you risk getting rejected transactions every time a user changes their Scatter Identity.
 
 
#### Catching browsers with Scatter installed
```
document.addEventListener('scatterLoaded', scatterExtension => { 
    const version = scatterExtension.version;
    
    // This will only be given in the case of previous Identity permissions.
    const hash = scatterExtension.identifier;
    
    //...
})
```

#### Initializing the Scatter interface

```
const scatter = window.scatter;
 
// It is good practice to take this off the window now.
window.scatter = null;
 
// Set up the network and options you want to use eosjs and Scatter with. 
const network = { host:"192.168.56.101", port:8888 };
const eosOptions = {};
 
// Tell Scatter to prepare and instance of eosjs with a 
// Scatter Signature Provider pre-configured. There is no way for
// you to use a custom built eosjs with a Scatter provider.
const eos = scatter.eos( Eos.Localnet, network, eosOptions );
```


#### Requesting an Identity

Once an Identity is provided it will not need to be re-approved every time. 
**You should re-request the identity before signature requests to make sure it has
not changed**. Check the flow diagram for more details.

```
// You can require certain fields
const requirements = ['account'];
 
scatter.getIdentity(requirements).then(identity => {
 
    // Identities must be bound to scatter to be 
    // able to request transaction signatures
    scatter.useIdentity(identity.hash);
    
}).catch(error => {
    //...
});
```

##### Fields that can be required
- **account** ( needs to be required for signature requests )
- firstname
- lastname
- email
- birthdate
- address
- city
- state
- country
- zipcode


#### Requesting a Signature
``` 
eos.transfer(identity.account.name, 'inita', 10, '').then(transaction => {
    //...
}).catch(error => { 
    //.. 
});
```
That's it! There is no difference using eosjs through Scatter than using eosjs with your own provider.
The only thing that changes is the **user's** flow.

#### Transactions at the Identity
```
// Standard inita key
const keyProvider = '5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3';
 
// Create your own instance of eosjs with your keyProvider and network
let customEos = Eos.Localnet({httpEndpoint:`http://${network.host}:${network.port}`, keyProvider});
 
// The same process as before but now you own the keys.
customEos.transfer('inita', identity.account.name, 100000, '').then(transaction => {
    //...
}).catch(error => { 
    //... 
});
```