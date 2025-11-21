//
// CodeSignVerify.swift
// Demonstrates signing and verifying files with CryptoKit (Ed25519)
// Swift 5+, CryptoKit available on macOS/iOS
//

import Foundation
import CryptoKit

// Generate keypair, sign a file, and verify
struct CodeSigner {
    let privateKey: Curve25519.Signing.PrivateKey
    let publicKey: Curve25519.Signing.PublicKey
    
    init() {
        self.privateKey = Curve25519.Signing.PrivateKey()
        self.publicKey = privateKey.publicKey
    }
    
    func signFile(url: URL) throws -> Data {
        let data = try Data(contentsOf: url)
        let signature = try privateKey.signature(for: data)
        return signature
    }
    
    static func verify(signature: Data, url: URL, publicKey: Curve25519.Signing.PublicKey) throws -> Bool {
        let data = try Data(contentsOf: url)
        return publicKey.isValidSignature(signature, for: data)
    }
}

// Demo usage
let tmpDir = FileManager.default.temporaryDirectory
let fileURL = tmpDir.appendingPathComponent("sample.txt")
let msg = "Important file content at \(Date())\n"
try? msg.data(using: .utf8)?.write(to: fileURL)

let signer = CodeSigner()
do {
    let sig = try signer.signFile(url: fileURL)
    print("Signature (hex):", sig.map { String(format: "%02x", $0) }.joined())
    let ok = try CodeSigner.verify(signature: sig, url: fileURL, publicKey: signer.publicKey)
    print("Verification result:", ok ? "VALID" : "INVALID")
} catch {
    print("Error signing/verifying:", error)
}
