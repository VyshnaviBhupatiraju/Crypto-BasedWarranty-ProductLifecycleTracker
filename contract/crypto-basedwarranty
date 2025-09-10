// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

/**
 * @title Crypto-Based Warranty Product Lifecycle Tracker
 * @dev A smart contract that manages product warranties and lifecycle tracking on blockchain
 * @author Smart Contract Developer
 */
contract Project {
    
    // Enums
    enum ProductStatus { Manufactured, Sold, InWarranty, WarrantyExpired, Serviced, Recalled, Disposed }
    enum ClaimStatus { Pending, Approved, Rejected, Completed }
    
    // Structures
    struct Product {
        uint256 productId;
        string productName;
        string model;
        string serialNumber;
        address manufacturer;
        address currentOwner;
        uint256 manufacturingDate;
        uint256 saleDate;
        uint256 warrantyPeriod; // in seconds
        ProductStatus status;
        bool exists;
    }
    
    struct WarrantyClaim {
        uint256 claimId;
        uint256 productId;
        address claimant;
        string issueDescription;
        uint256 claimDate;
        uint256 resolutionDate;
        ClaimStatus status;
        string resolution;
        uint256 serviceCost;
    }
    
    struct ServiceRecord {
        uint256 serviceId;
        uint256 productId;
        address serviceProvider;
        uint256 serviceDate;
        string serviceDescription;
        uint256 cost;
        string partsReplaced;
    }
    
    // State variables
    address public admin;
    uint256 public nextProductId;
    uint256 public nextClaimId;
    uint256 public nextServiceId;
    
    // Mappings
    mapping(uint256 => Product) public products;
    mapping(uint256 => WarrantyClaim) public warrantyClaims;
    mapping(uint256 => ServiceRecord) public serviceRecords;
    mapping(address => bool) public authorizedManufacturers;
    mapping(address => bool) public authorizedServiceProviders;
    mapping(uint256 => uint256[]) public productClaims; // productId => claimIds[]
    mapping(uint256 => uint256[]) public productServices; // productId => serviceIds[]
    mapping(string => uint256) public serialToProductId; // serialNumber => productId
    
    // Events
    event ProductRegistered(uint256 indexed productId, string serialNumber, address indexed manufacturer);
    event ProductSold(uint256 indexed productId, address indexed newOwner, uint256 saleDate);
    event WarrantyClaimSubmitted(uint256 indexed claimId, uint256 indexed productId, address indexed claimant);
    event ClaimStatusUpdated(uint256 indexed claimId, ClaimStatus status);
    event ServiceRecorded(uint256 indexed serviceId, uint256 indexed productId, address indexed serviceProvider);
    event ProductStatusUpdated(uint256 indexed productId, ProductStatus status);
    
    // Modifiers
    modifier onlyAdmin() {
        require(msg.sender == admin, "Only admin can perform this action");
        _;
    }
    
    modifier onlyManufacturer() {
        require(authorizedManufacturers[msg.sender], "Only authorized manufacturers allowed");
        _;
    }
    
    modifier onlyServiceProvider() {
        require(authorizedServiceProviders[msg.sender], "Only authorized service providers allowed");
        _;
    }
    
    modifier productExists(uint256 _productId) {
        require(products[_productId].exists, "Product does not exist");
        _;
    }
    
    modifier onlyProductOwner(uint256 _productId) {
        require(products[_productId].currentOwner == msg.sender, "Only product owner can perform this action");
        _;
    }
    
    // Constructor
    constructor() {
        admin = msg.sender;
        nextProductId = 1;
        nextClaimId = 1;
        nextServiceId = 1;
    }
    
    /**
     * @dev Core Function 1: Register a new product with warranty details
     * @param _productName Name of the product
     * @param _model Product model
     * @param _serialNumber Unique serial number
     * @param _warrantyPeriod Warranty period in seconds
     */
    function registerProduct(
        string memory _productName,
        string memory _model,
        string memory _serialNumber,
        uint256 _warrantyPeriod
    ) external onlyManufacturer {
        require(bytes(_serialNumber).length > 0, "Serial number cannot be empty");
        require(serialToProductId[_serialNumber] == 0, "Product with this serial number already exists");
        require(_warrantyPeriod > 0, "Warranty period must be greater than 0");
        
        uint256 productId = nextProductId++;
        
        products[productId] = Product({
            productId: productId,
            productName: _productName,
            model: _model,
            serialNumber: _serialNumber,
            manufacturer: msg.sender,
            currentOwner: msg.sender,
            manufacturingDate: block.timestamp,
            saleDate: 0,
            warrantyPeriod: _warrantyPeriod,
            status: ProductStatus.Manufactured,
            exists: true
        });
        
        serialToProductId[_serialNumber] = productId;
        
        emit ProductRegistered(productId, _serialNumber, msg.sender);
    }
    
    /**
     * @dev Core Function 2: Submit warranty claim for a product
     * @param _productId Product ID for the claim
     * @param _issueDescription Description of the issue
     */
    function submitWarrantyClaim(
        uint256 _productId,
        string memory _issueDescription
    ) external productExists(_productId) onlyProductOwner(_productId) {
        Product storage product = products[_productId];
        require(product.saleDate > 0, "Product not yet sold");
        require(isUnderWarranty(_productId), "Product warranty has expired");
        require(bytes(_issueDescription).length > 0, "Issue description cannot be empty");
        
        uint256 claimId = nextClaimId++;
        
        warrantyClaims[claimId] = WarrantyClaim({
            claimId: claimId,
            productId: _productId,
            claimant: msg.sender,
            issueDescription: _issueDescription,
            claimDate: block.timestamp,
            resolutionDate: 0,
            status: ClaimStatus.Pending,
            resolution: "",
            serviceCost: 0
        });
        
        productClaims[_productId].push(claimId);
        
        emit WarrantyClaimSubmitted(claimId, _productId, msg.sender);
    }
    
    /**
     * @dev Core Function 3: Record service/repair for a product
     * @param _productId Product ID being serviced
     * @param _serviceDescription Description of service performed
     * @param _cost Cost of service
     * @param _partsReplaced Parts that were replaced
     */
    function recordService(
        uint256 _productId,
        string memory _serviceDescription,
        uint256 _cost,
        string memory _partsReplaced
    ) external productExists(_productId) onlyServiceProvider {
        require(bytes(_serviceDescription).length > 0, "Service description cannot be empty");
        
        uint256 serviceId = nextServiceId++;
        
        serviceRecords[serviceId] = ServiceRecord({
            serviceId: serviceId,
            productId: _productId,
            serviceProvider: msg.sender,
            serviceDate: block.timestamp,
            serviceDescription: _serviceDescription,
            cost: _cost,
            partsReplaced: _partsReplaced
        });
        
        productServices[_productId].push(serviceId);
        
        // Update product status to serviced
        products[_productId].status = ProductStatus.Serviced;
        
        emit ServiceRecorded(serviceId, _productId, msg.sender);
        emit ProductStatusUpdated(_productId, ProductStatus.Serviced);
    }
    
    // Administrative functions
    
    /**
     * @dev Add authorized manufacturer
     * @param _manufacturer Address of manufacturer to authorize
     */
    function addManufacturer(address _manufacturer) external onlyAdmin {
        require(_manufacturer != address(0), "Invalid manufacturer address");
        authorizedManufacturers[_manufacturer] = true;
    }
    
    /**
     * @dev Add authorized service provider
     * @param _serviceProvider Address of service provider to authorize
     */
    function addServiceProvider(address _serviceProvider) external onlyAdmin {
        require(_serviceProvider != address(0), "Invalid service provider address");
        authorizedServiceProviders[_serviceProvider] = true;
    }
    
    /**
     * @dev Transfer product ownership (sale)
     * @param _productId Product ID to transfer
     * @param _newOwner New owner address
     */
    function transferOwnership(uint256 _productId, address _newOwner) external productExists(_productId) {
        require(msg.sender == products[_productId].manufacturer || msg.sender == products[_productId].currentOwner, "Not authorized to transfer");
        require(_newOwner != address(0), "Invalid new owner address");
        
        Product storage product = products[_productId];
        product.currentOwner = _newOwner;
        
        if (product.saleDate == 0) {
            product.saleDate = block.timestamp;
            product.status = ProductStatus.InWarranty;
        }
        
        emit ProductSold(_productId, _newOwner, block.timestamp);
        emit ProductStatusUpdated(_productId, product.status);
    }
    
    /**
     * @dev Update warranty claim status (manufacturer only)
     * @param _claimId Claim ID to update
     * @param _status New claim status
     * @param _resolution Resolution description
     * @param _serviceCost Cost of service if applicable
     */
    function updateClaimStatus(
        uint256 _claimId,
        ClaimStatus _status,
        string memory _resolution,
        uint256 _serviceCost
    ) external {
        WarrantyClaim storage claim = warrantyClaims[_claimId];
        require(claim.claimId != 0, "Claim does not exist");
        
        Product memory product = products[claim.productId];
        require(msg.sender == product.manufacturer, "Only manufacturer can update claim status");
        
        claim.status = _status;
        claim.resolution = _resolution;
        claim.serviceCost = _serviceCost;
        
        if (_status == ClaimStatus.Completed) {
            claim.resolutionDate = block.timestamp;
        }
        
        emit ClaimStatusUpdated(_claimId, _status);
    }
    
    // View functions
    
    /**
     * @dev Check if product is under warranty
     * @param _productId Product ID to check
     * @return bool Whether product is under warranty
     */
    function isUnderWarranty(uint256 _productId) public view productExists(_productId) returns (bool) {
        Product memory product = products[_productId];
        if (product.saleDate == 0) return false;
        return (block.timestamp <= product.saleDate + product.warrantyPeriod);
    }
    
    /**
     * @dev Get warranty expiration date
     * @param _productId Product ID
     * @return uint256 Warranty expiration timestamp
     */
    function getWarrantyExpirationDate(uint256 _productId) external view productExists(_productId) returns (uint256) {
        Product memory product = products[_productId];
        if (product.saleDate == 0) return 0;
        return product.saleDate + product.warrantyPeriod;
    }
    
    /**
     * @dev Get product by serial number
     * @param _serialNumber Serial number to lookup
     * @return Product Product details
     */
    function getProductBySerial(string memory _serialNumber) external view returns (Product memory) {
        uint256 productId = serialToProductId[_serialNumber];
        require(productId != 0, "Product not found");
        return products[productId];
    }
    
    /**
     * @dev Get all claims for a product
     * @param _productId Product ID
     * @return uint256[] Array of claim IDs
     */
    function getProductClaims(uint256 _productId) external view productExists(_productId) returns (uint256[] memory) {
        return productClaims[_productId];
    }
    
    /**
     * @dev Get all service records for a product
     * @param _productId Product ID
     * @return uint256[] Array of service IDs
     */
    function getProductServices(uint256 _productId) external view productExists(_productId) returns (uint256[] memory) {
        return productServices[_productId];
    }
    
    /**
     * @dev Get complete product lifecycle information
     * @param _productId Product ID
     * @return product Product details
     * @return claimIds Array of claim IDs
     * @return serviceIds Array of service IDs
     * @return warrantyActive Whether warranty is active
     */
    function getProductLifecycle(uint256 _productId) external view productExists(_productId) returns (
        Product memory product,
        uint256[] memory claimIds,
        uint256[] memory serviceIds,
        bool warrantyActive
    ) {
        product = products[_productId];
        claimIds = productClaims[_productId];
        serviceIds = productServices[_productId];
        warrantyActive = isUnderWarranty(_productId);
    }
}
##contract
"C:\Users\vskv8\OneDrive\Pictures\Screenshots\Screenshot 2025-09-10 140330.png"
