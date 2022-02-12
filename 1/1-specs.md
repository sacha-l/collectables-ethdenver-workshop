# What we're building

The pallet we're building is based on the ERC721 standard.
Here are some of the ERC721 features our pallet will have.

From ERC721 interface:

* `function balanceOf(_owner)-> uint256`: returns the number of NFTs owned by `_owner`.
* `function ownerOf(_tokenId)-> address`: returns the address of the owner of the NFT. 
* `function safeTransferFrom(_from, _to, _tokenId)`: transfers ownership of NFT.

From the ERC721Enumerable interface:

* `function totalSupply()`: returns the total supply of existing NFTs.
* `function tokenByIndex(_index)`: returns the identifier for an NFT at some index.
* `function tokenOfOwnerByIndex(_owner, _index)`: returns the identifier of the NFTs assigned to some owner

Our pallet is pretty much analagous.
But we're going to build a **specific type of NFT representing the DNA of a collectable Kitty.** 
It'll show how we're able to build some NFT-like standard and tell a well known story around it, the CryptoKitties story. 
With pallets,  we're able to customize and extend the logic of our NFT application in so many ways.. 
Extra points to those of you who create the Bufficorn front-end for the chain we're building!