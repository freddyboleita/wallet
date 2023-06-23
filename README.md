
# WALLETS

Los servicios tipo  `Wallet` tales como los de Google y Apple permiten crear almacenar documentos de manera virtual. Como su nombre lo indica cumplen la funcion de una billetera virtual en la cual puedes cargar documentos y tarjetas para hacer pagos.

## GOOGLE WALLET

Google Wallet c permite crear lases de ticketes para eventos y crear objetos de esa clase. Tambien permite almacenarlos en la cuenta propia para ser usados posteriormente.
Para la interaccion con este servicio se tiene el `API` en la cual se pueden hacer peticiones para hacer uso de los mismos mediante una aplicacion.

En este repositorio se encuentra un ejemplo usando Go y otro usando Node para crear una clase y un objeto de esa clase.
El ejemplo de Node se hizo siguiendo la guia [codelab google wallet](https://g.co/wallet/web-codelab), en el caso de Node se tomo del repositorio [wallet](https://github.com/google-pay/wallet-samples). En ese repositorio se encuentran ejemplos de `Go`, `Java`, `Python`, etc.

En resumen se puede integrar con practicamente cualquier lenguaje debido a que tiene una API publica la cual se puede usar, el unico pre-requisito es tener una cuenta de Google como desarrollador. La creacion de la cuenta de Google se especifica en la guia anterior mente mencionada.

Addicional a la informacion ya dada se deja la referencia a la [`API`](https://developers.google.com/wallet/tickets/events/web/prerequisites) donde se encuentra toda la informacion para poder interactuar con `Google Wallet`


### Integracion

Se definen las url base a las cuales le haremos peticiones

```Go
const (
  batchUrl  = "https://walletobjects.googleapis.com/batch"
  classUrl  = "https://walletobjects.googleapis.com/walletobjects/v1/eventTicketClass"
  objectUrl = "https://walletobjects.googleapis.com/walletobjects/v1/eventTicketObject"
)
```

Todas estas url reciben un json como cuerpo que para efectos de esta prueba de concepto se envian como un  `string`.

La autenticacion se hace con oauth2 

```Go
func (d *demoEventticket) auth() {
	b, _ := os.ReadFile(os.Getenv("GOOGLE_APPLICATION_CREDENTIALS"))
	credentials, _ := google.JWTConfigFromJSON(b, "https://www.googleapis.com/auth/wallet_object.issuer")
	d.credentials = credentials
	d.httpClient = d.credentials.Client(context.TODO())
}
```


Se la variable de entorno `GOOGLE_APPLICATION_CREDENTIALS` que debe contener la ruta al archivo con las credenciales para acceder a Google. Esto esta explicado de manera completa en la guia dejada inicialmente

Con esta funcion se crea un cliente http autenticado con nuestras credenciales

Con estos pasos ya hechos se puede crear una clase de ticket usando la siguiente funcion.

```Go

func (d *demoEventticket) createClass(issuerId, classSuffix string) {
  newClass := fmt.Sprintf(`
  {
    "eventId": "EVENT_ID",
    "eventName": {
      "defaultValue": {
        "value": "Event name",
        "language": "en-US"
      }
    },
    "issuerName": "Issuer name",
    "id": "%s.%s",
    "reviewStatus": "UNDER_REVIEW"
  }
  `, issuerId, classSuffix)

  res, err := d.httpClient.Post(classUrl, "application/json", bytes.NewBuffer([]byte(newClass)))

  if err != nil {
    fmt.Println(err)
  } else {
    b, _ := io.ReadAll(res.Body)
    fmt.Printf("Class insert response:\n%s\n", b)
  }
}
```

En esta funcionse envia el `JSON` newClass con el cual crearemos nuestra clase de ticket en Google. Cabe aclarar que todas estas peticiones van autenticadas con el cliente creado anteriormente.

Luego de esto podemos proceder a crear un obejto de la clase `EVENT_ID`

```Go
func (d *demoEventticket) createObject(issuerId, classSuffix, objectSuffix string) {
  newObject := fmt.Sprintf(`
  {
    "classId": "%s.%s",
    "ticketHolderName": "Ticket holder name",
    "heroImage": {
      "contentDescription": {
        "defaultValue": {
          "value": "Hero image description",
          "language": "en-US"
        }
      },
      "sourceUri": {
        "uri": "https://farm4.staticflickr.com/3723/11177041115_6e6a3b6f49_o.jpg"
      }
    },
    "barcode": {
      "type": "QR_CODE",
      "value": "QR code"
    },
    "locations": [
      {
        "latitude": 37.424015499999996,
        "longitude": -122.09259560000001
      }
    ],
    "state": "ACTIVE",
    "linksModuleData": {
      "uris": [
        {
          "id": "LINK_MODULE_URI_ID",
          "uri": "http://maps.google.com/",
          "description": "Link module URI description"
        },
        {
          "id": "LINK_MODULE_TEL_ID",
          "uri": "tel:6505555555",
          "description": "Link module tel description"
        }
      ]
    },
    "ticketNumber": "Ticket number",
    "imageModulesData": [
      {
        "id": "IMAGE_MODULE_ID",
        "mainImage": {
          "contentDescription": {
            "defaultValue": {
              "value": "Image module description",
              "language": "en-US"
            }
          },
          "sourceUri": {
            "uri": "http://farm4.staticflickr.com/3738/12440799783_3dc3c20606_b.jpg"
          }
        }
      }
    ],
    "textModulesData": [
      {
        "body": "Text module body",
        "header": "Text module header",
        "id": "TEXT_MODULE_ID"
      }
    ],
    "seatInfo": {
      "gate": {
        "defaultValue": {
          "value": "A",
          "language": "en-US"
        }
      },
      "section": {
        "defaultValue": {
          "value": "5",
          "language": "en-US"
        }
      },
      "row": {
        "defaultValue": {
          "value": "G3",
          "language": "en-US"
        }
      },
      "seat": {
        "defaultValue": {
          "value": "42",
          "language": "en-US"
        }
      }
    },
    "id": "%s.%s"
  }
  `, issuerId, classSuffix, issuerId, objectSuffix)

  res, err := d.httpClient.Post(objectUrl, "application/json", bytes.NewBuffer([]byte(newObject)))

  if err != nil {
    fmt.Println(err)
  } else {
    b, _ := io.ReadAll(res.Body)
    fmt.Printf("Object insert response:\n%s\n", b)
  }
}
```

En esta funcion se crea un objeto de la clase que creamos anteriormente con la informacion enviada en el string `newObject`.

La validacion de la creacion de este 
## APPLE WALLET

Al igual que Google Wallet, el servicio de Apple permite almacenar tickets para eventos y tarjedas de credito para hacer pagos mediante NFC.
Para poder hacer una prueba de concepto se debe tener una cuenta que tenga la membresia de desarrollador, esto debido a que si tendremos un ID o certificado con el cual acceder a la `API` de Apple Wallet.

Para crear el ID se puede seguir este [paso a paso](https://developer.apple.com/help/account/configure-app-capabilities/create-order-type-identifiers-and-certificates/)

Apple nos provee con una [guia](https://developer.apple.com/wallet/get-started/) para poder usar este servicio.

Desgraciadamente para hacer uso de estos servicios se debe comprar una licencia 