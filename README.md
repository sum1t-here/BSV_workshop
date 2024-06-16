deploy.ts

```javacript
import { DefaultProvider, sha256, bsv, toByteString } from 'scrypt-ts'
import { Demo } from './src/contracts/demo'
import { NeucronSigner } from 'neucron-signer'

async function main() {
    const provider = new DefaultProvider({ network: bsv.Networks.mainnet })
    const signer = new NeucronSigner(provider)
    const amount = 1

    await signer.login('sales@timechainlabs.io', 'string')
    await Demo.loadArtifact()

    const message = toByteString('timechainlabs', true)
    const instance = new Demo(sha256(message))
    await instance.connect(signer)

    const deployTx = await instance.deploy(amount)
    console.log(
        'smart lock deployed : https://whatsonchain.com/tx/' + deployTx.id
    )

    await new Promise((f) => setTimeout(f, 5000))
    const { tx: callTx } = await instance.methods.unlock(message)
    console.log(
        'contract unlocked successfully : https://whatsonchain.com/tx/' +
            callTx.id
    )
}

main()
```
```javascript
src/contracts
import {
    ByteString,
    Sha256,
    sha256,
    SmartContract,
    assert,
    method,
    prop,
} from 'scrypt-ts'

export class Demo extends SmartContract {
    @prop()
    hash: Sha256

    constructor(hash: Sha256) {
        super(...arguments)
        this.hash = hash
    }

    @method()
    public unlock(message: ByteString) {
        assert(sha256(message) == this.hash, 'incorrect hash')
    }
}
```