# Extension History

Our history extension is capable of taking snapshots of a yjs document automatically or on command,
and can revert back the existing document to any older version.

## Installation

:::warning Pro Feature
To get started, you need a Tiptap Pro account ([sign up / login here](https://tiptap.dev/pro)).
:::

Install the History extension like this:

```bash
npm install @hocuspocus-pro/extension-history
```

## Configuration

**isActive** can be used to enable/disable the extension during runtime. You can pass a boolean or an async callback that resolves to a boolean.

**onVersionCreated** this is called after a version was created. The passed parameters should be stored on a persistent storage. The `stateAsUpdate` and `versionNumber` fields are needed when restoring an old version.

**getVersion** this is called when an old version is requested. It should return the `stateAsUpdate` field.

## Usage (frontend)

```typescript
provider.createVersion({ name: 'my first version'} )
provider.revertToVersion(0)

provider.isAutoVersioning()

provider.enableAutoVersioning()
provider.disableAutoVersioning()

provider.watchVersions()
provider.getVersions()
```


## Usage (backend)
```typescript
const server = Server.configure({
  extensions: [
    new History({
      isActive: true,
      getVersion: async (documentName, versionNumber) => {
        const document = await prisma.document.findFirst({ where: { name: documentName } })

        if ( !document ) {
          throw new Error(`Document ${documentName} not found.`)
        }

        const version = await prisma.documentVersion.findFirst({
          where: {
            documentId: document.id,
            version: versionNumber,
          },
        })

        if ( !version ) {
          throw new Error(`Version ${versionNumber} for document ${documentName} not found.`)
        }

        return version.stateAsUpdate
      },
      onVersionCreated: async (documentName, versionName, versionNumber, stateAsUpdate) => {
        const document = await prisma.document.findFirst({ where: { name: documentName } })

        if ( !document ) {
          throw new Error(`Document ${documentName} not found.`)
        }

        return prisma.documentVersion.create({
          data: {
            stateAsUpdate: Buffer.from(stateAsUpdate),
            version: versionNumber,
            documentId: document.id,
          },
        })
      },
    }),
  ],
})
```
