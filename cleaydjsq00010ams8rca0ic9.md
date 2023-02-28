# #100DaysofCode - Day 1

*Day 1 of the 100 Days of Code challenge*

For the next 100 days, I will be dedicating myself to improving my coding skills and working on various projects. Today, I focused on learning and working with Solidity, a programming language used to develop smart contracts on the Ethereum blockchain. In this article, I will outline my progress and share some of my experiences as I begin my coding journey. Whether you're a seasoned developer or just starting out, I hope you'll find this article both informative and inspiring.

I was inspired mostly by @[Tanisha Chandani](@tanishachandani) to start this challenge and I figured there was no better day than a Saturday. I was lounging around on a warm Oakland afternoon when I found the motivation to pick up my Solidity project and start programming.

For development, I am using the Remix IDE due to its simplicity and tool set offering for developing Solidity contracts. Truly the swiss knife of blockchain development as it is the most popular development environment for writing and testing smart contracts on the Ethereum blockchain. It is an online tool that provides developers with a user-friendly interface for creating and deploying Solidity smart contracts. Remix comes equipped with a suite of features, including a code editor, compiler, debugger, and testing environment, that enable developers to write, test, and debug their contracts in a single platform. Additionally, Remix has built-in integrations with various Ethereum tools, such as MetaMask and Geth, which makes it easy for developers to deploy their contracts directly from the IDE. Overall, Remix is a powerful and flexible IDE that provides a comprehensive solution for developing and deploying smart contracts on Ethereum.

Want to know more about getting started with Remix? Check out some of these tutorials below.

### Chainlink + Free Code Camp

%[https://www.youtube.com/watch?v=5dcRMHUhA20] 

### Eat The Blocks

%[https://www.youtube.com/watch?v=WmeWbo7wzGI] 

### Free Code Camp

%[https://www.youtube.com/watch?v=ipwxYa-F1uY&t=191s] 

These are great tutorials to get a grasp of getting started with smart contract development with the Remix IDE. All of these videos are from reputable people/entities in the blockchain space and they all have a lot of great content to learn from on their channels.

I won't reveal the whole project at once but I will explain the pieces I was working on today.

Tasks Accomplished

* Create a role-based access control mechanism that will assign roles to users with NFT access passes
    
    * The user who deploys the contract is the first Admin for the system
        
    * Only Admins are allowed to assign and revoke roles from users
        
* Create NFT Access Passes for users that act as an account
    
    * There can be many account owners
        
    * The NFT can hold account information in memory
        
    * The account has many policies
        
    * The account has many items to be insured
        
* Create an NFT that represents an Insurance Policy
    
    * This policy should be able to hold value in tokens
        
    * This policy should have many owners
        
    * This policy is able to payout its value to owners
        
    * This policy has policy information and coverages
        
* Create an NFT that represents the Insured Item
    

### First things first

Let's get the UserAccount contract out of the way. With this contract, it sets the basis of access control for the platform. Users will navigate through the platform with this NFT and will gain access to different parts of the UI depending on their mapped role. We will get to the role access later. First let's make this NFT.

```solidity
contract UserAccountNFT is IERC721, IERC1155, Ownable {
    struct MailingAddress {
        string street;
        string city;
        string state;
        uint256 postalCode;
        string country;
        string propertyType;
    }

        struct Person {
        uint256 personId;
        string profilePhoto;
        string firstName;
        string lastName;
        uint256 birthdate;
        MailingAddress mailingAddress;
        string email;
        string phoneNumber;
        bytes32 role;
    }

    mapping (uint256 => address) private _owners;
    mapping (address => uint256) private _balances;
    mapping (uint256 => uint256) private _policiesOwned;
    mapping (uint256 => uint256[]) private _petsOwned;
    mapping (uint256 => bytes) private _personData;

    Person internal accountInfo;
    IERC20 public erc20Token;
    IERC721 public policyNFT;
    IERC721 public petNFT;

    constructor(IERC20 _erc20Token, IERC721 _policyNFT, IERC721 _petNFT) {
        erc20Token = _erc20Token;
        policyNFT = _policyNFT;
        petNFT = _petNFT;
    }

    function balanceOf(address owner, uint256 id) public view override returns (uint256) {
        if (_owners[id] == owner) {
            return 1;
        } else {
            return 0;
        }
    }

    function ownerOf(uint256 id) public view override returns (address) {
        return _owners[id];
    }

    function safeTransferFrom(address from, address to, uint256 id, uint256 amount, bytes memory data) public override {
        if (_owners[id] == from) {
            if (amount == 1) {
                _owners[id] = to;
            } else {
                revert("UserAccount: amount must be 1");
            }
        } else {
            revert("UserAccount: sender is not owner");
        }
    }

    function transferFrom(address from, address to, uint256 id) public override {
        safeTransferFrom(from, to, id, 1, "");
    }

    function mint(uint256 id, address initialOwner) public {
        _owners[id] = initialOwner;
        _balances[initialOwner]++;
    }

    function burn(uint256 id, uint256 amount) public override {
        require(_owners[id] == msg.sender, "UserAccount: sender is not owner");
        super.burn(id, amount);
    }

    function approve(address to, uint256 id) public override {
        require(_owners[id] == msg.sender, "UserAccount: sender is not owner");
        super.approve(msg.sender, to, id);
    }
        
    function setApprovalForAll(address operator, bool approved) public override {}

    function isApprovedForAll(address owner, address operator) public view override returns (bool) {}

    function setAccountInfo(uint256 tokenId, Person memory person) public onlyOwner {
        accountInfo = person;
        _personData[tokenId] = abi.encode(person);
    }

    function getOwner(uint256 ownerId) public view returns (Person memory) {
        return _owners[ownerId];
    }

    function getAccountData(uint256 accountId) public view returns (bytes memory) onlyOwner{
        return _personData[accountId];
    }

    function setPolicy(uint256 accountId, uint256 policyTokenId, address constructionAddress) public onlyOwner {
        policyNFT.safeTransferFrom(constructionAddress, address(this), policyTokenId);
        _policiesOwned[accountId] = policyTokenId;
    }

    function setItem(uint256 accountId, uint256 itemTokenId, address constructionAddress) public onlyOwner {
        itemNFT.safeTransferFrom(constructionAddress, address(this), itemTokenId);
        _itemsOwned[accountId].push(itemTokenId);
    }
}
```

It is still a work in progress but it covers the basics of being an access tool. Creating a relationship between owned policies and items will allow us to access data quickly. Let me know your thoughts on the implementation!

### Give it a purpose

The next piece I want to highlight is granting a user role-based access to the system. I want to be able to control how the contract gives access to users and be able to retain the mappings in the contract as well. That way, we can limit who has access to that data and who can manipulate that data.

We will have 4 roles:

```solidity
contract RoleConstants{
    bytes32 public constant ADMIN_ROLE = keccak256("ADMIN_ROLE");
    bytes32 public constant POLICY_HOLDER_ROLE = keccak256("POLICY_HOLDER_ROLE");
    bytes32 public constant SHAREHOLDER_ROLE = keccak256("SHAREHOLDER_ROLE");
    bytes32 public constant CLAIM_COORDINATOR_ROLE = keccak256("CLAIM_COORDINATOR_ROLE");
}
```

As you can tell by now, the platform is intended to be an insurance platform of sorts. In traditional insurance companies, there are many roles that members apart of the company perform. Simplifying that list, we are left with roles for users that satisfy users that hold a policy, help with claims, fund the project, and run the whole show.

Defining the role names and a unique byte sequence to define each of them is just the beginning.

Now let's use them.

```solidity
contract PlatformAccessControl is AccessControl {

    // Define mapping of roles to NFTs
    mapping(bytes32 => IERC721) private privilegedUsers;

    // Define events for role assignment and revocation
    event RoleAssigned(bytes32 indexed role, address indexed account, uint256 indexed accountId);
    event RoleRevoked(bytes32 indexed role, address indexed account, uint256 indexed accountId);



    constructor() {
        _setupRole(RoleConstants.ADMIN_ROLE, msg.sender);
    }

    function grantRole(bytes32 role, address account, uint256 accountId) public onlyRole(RoleConstants.ADMIN_ROLE) {
        revert(privilegedUsers[accountId])
        // Check if account already has the role
        if (hasRole(role, account)) {
            revokeRole(role, account, accountId);
        }

        super.grantRole(role, account);
        emit RoleAssigned(role, account, accountId);
    }

    function revokeRole(bytes32 role, address account, uint256 accountId) public onlyRole(RoleConstants.ADMIN_ROLE) {
        // Check if account has the role
        if (!hasRole(role, account)) {
            return;
        }

        super.revokeRole(role, account);
        emit RoleRevoked(role, account, accountId);
    }

    function setNftRole(uint256 accountId, bytes32 role) public onlyRole(RoleConstants.ADMIN_ROLE) {
        _privilegedUsers[accountId] = role;
    }

    function getNftRole(uint256 accountId) public view returns (bytes32) {
        return _privilegedUsers[accountId];
    }

}
```

Will be back tomorrow with some more progress on this and an Ionic app :)

All feedback is welcomed!