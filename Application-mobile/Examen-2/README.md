# Examen Mobile 1

## Ajout de style personnalisé

```swift
//content view
 Rectangle()
    .fill(Color.blue)
    .modifier(RectangleCustomStyle())
    //OU
    .customRectangle()
```

```swift
// Bas de page
extension View {
  func customRectangle() -> some View {
    modifier(RectangleCustomStyle())
  }
}

struct RectangleCustomStyle: ViewModifier {
    func body(content: Content) -> some View {
        content
            .frame(width: 50, height: 70)
            .border(Color.black, width: 5)
            .zIndex(1)
      }
}
```


## Page de base pour BD

ContentView de base

```swift
struct ContentView: View {
    
    @State var gestionBD: GestionBD = GestionBD(nomBD: "rectangles.db");
    @State var initialDonne : Bool = false
    
    init() {
        gestionBD.ouvrirBD()
    }
    
    var body: some View {
        NavigationView{       
            VStack{
                 if gestionBD.pointeurBD == nil {
                    Text("Un problème empêche l'ouverture de la base de données.")
                } else {
                    //CODE ICI             
                }
            }
            .toolbar(content: {
                ToolbarItem(placement: .principal, content: {
                    HStack(){
                        Text("Jonathan Côté")
                    }
                    
                })
            })
        }.onAppear {
            if(!initialDonne){
                initialDonne = true
            }
        }
    }    
}
```  

## gestion.bd
    
```swift
//
//  GestionBD.swift
//  GestionObjetConnecte
//
//  Code des note de cours BD apical 
//

import Foundation
import SQLite3
import SwiftUI

/// Gestion d'une base de données SQLite.
/// Code des notes de cours BD apical
class GestionBD {
  var nomBD: String = ""
  var pointeurBD: OpaquePointer? = nil

  /**
   Constructeur.

   - Parameters:
     - nomBD: nom de la base de données
  */
  init(nomBD: String) {
    self.nomBD = nomBD
  }    

   /**
    Ouvre la base de données et initialise la propriété pointeurBD.
   */
  func ouvrirBD() {
    var reussi: Bool = false

    do {
      let fileManager = FileManager.default

      let urlApplicationSupport = try fileManager.url(for: .applicationSupportDirectory, in: .userDomainMask, appropriateFor: nil, create: true)
        .appendingPathComponent(nomBD)
      // print(urlApplicationSupport)

      // ouvre la base de données mais ne la crée pas si elle n'existe pas
      var codeRetour = sqlite3_open_v2(urlApplicationSupport.path, &pointeurBD, SQLITE_OPEN_READWRITE, nil)

      // si la base de données n'est pas trouvée, va chercher la version originale dans le paquet de l'application
      if codeRetour == SQLITE_CANTOPEN {
        print("Tentative de retrouver la base de données dans le paquet.")

        if let urlPaquet = Bundle.main.url(forResource: nomBD, withExtension: "") { // ici, l'extension fait déjà partie du nom de la BD
          // print(urlPaquet)

          try fileManager.copyItem(at: urlPaquet, to: urlApplicationSupport)
          codeRetour = sqlite3_open_v2(urlApplicationSupport.path, &pointeurBD, SQLITE_OPEN_READWRITE, nil)
        } else {
          print("Erreur : la base de données ne fait pas partie du paquet.")
        }
      }

      if codeRetour == SQLITE_OK {
        reussi = true;
      } else {
        print("La connexion à la base de données a échoué : \(codeRetour)")
      }

    } catch {
      print("Erreur inattendue : \(error)")
    }

    if (!reussi) {
      // Selon la doc officielle : Whether or not an error occurs when it is opened, resources associated with the database connection handle should be released by passing it to sqlite3_close() when it is no longer required (source : https://www.sqlite.org/c3ref/open.html).
      sqlite3_close(pointeurBD)
      pointeurBD = nil
    }
  }

    func insererResultat(resultat: Int) -> Bool {
        
      let requete: String = "INSERT INTO parties(date, points) VALUES(?, ?);"

      var retour = false;
      var preparation: OpaquePointer? = nil

      // prépare la requête
      if sqlite3_prepare_v2(pointeurBD, requete, -1, &preparation, nil) == SQLITE_OK {

          let date = Date.now

          // Create Date Formatter
          let dateFormatter = DateFormatter()

          // Convert Date to String
        
          sqlite3_bind_text(preparation, 1, NSString(string: dateFormatter.string(from: date)).utf8String, -1, nil)
          sqlite3_bind_int(preparation, 2, Int32(resultat))

        // exécute la requête
        if sqlite3_step(preparation) != SQLITE_DONE {
          let erreur = String(cString: sqlite3_errmsg(pointeurBD))
          print("\nErreur lors de l'insertion : \(erreur)")
        } else {
          retour = true
        }
      } else {
        let erreur = String(cString: sqlite3_errmsg(pointeurBD))
        print("\nLa requête n'a pas pu être préparée : \(erreur)")
      }

      // libération de la mémoire
      sqlite3_finalize(preparation)

      return retour
    }
}
```
