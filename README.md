
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

/**
 * @title AttendanceSystem
 * @dev A simple blockchain-based attendance tracking system
 */
contract AttendanceSystem {
    // Owner of the contract (typically the institution administrator)
    address public owner;
    
    // Structure to store attendance records
    struct Attendance {
        uint256 timestamp;   // When attendance was recorded
        string location;     // Physical or virtual location identifier
        string metadata;     // Additional information (can store encrypted data if needed)
    }
    
    // Structure to store class/event details
    struct Event {
        string name;         // Name of the class or event
        uint256 startTime;   // When the event starts
        uint256 endTime;     // When the event ends
        bool active;         // Whether the event is currently active
    }
    
    // Mapping from event ID to Event
    mapping(uint256 => Event) public events;
    
    // Mapping from event ID to participant address to Attendance record
    mapping(uint256 => mapping(address => Attendance)) public attendanceRecords;
    
    // List of registered participants (students, employees, etc.)
    mapping(address => bool) public registeredParticipants;
    
    // Total number of events created
    uint256 public eventCount;
    
    // Events
    event AttendanceMarked(uint256 indexed eventId, address indexed participant, uint256 timestamp);
    event EventCreated(uint256 indexed eventId, string name, uint256 startTime, uint256 endTime);
    event ParticipantRegistered(address indexed participant);
    
    // Constructor
    constructor() {
        owner = msg.sender;
    }
    
    // Modifiers
    modifier onlyOwner() {
        require(msg.sender == owner, "Only owner can call this function");
        _;
    }
    
    modifier onlyRegistered() {
        require(registeredParticipants[msg.sender], "Only registered participants can call this function");
        _;
    }
    
    /**
     * @dev Create a new event/class for attendance tracking
     * @param _name Name of the event
     * @param _startTime When the event starts
     * @param _endTime When the event ends
     * @return The ID of the newly created event
     */
    function createEvent(
        string memory _name,
        uint256 _startTime,
        uint256 _endTime
    ) public onlyOwner returns (uint256) {
        require(_endTime > _startTime, "End time must be after start time");
        
        eventCount++;
        uint256 eventId = eventCount;
        
        events[eventId] = Event({
            name: _name,
            startTime: _startTime,
            endTime: _endTime,
            active: true
        });
        
        emit EventCreated(eventId, _name, _startTime, _endTime);
        
        return eventId;
    }
    
    /**
     * @dev Mark attendance for a participant in a specific event
     * @param _eventId ID of the event
     * @param _location Physical or virtual location identifier
     * @param _metadata Additional information (can be encrypted)
     */
    function markAttendance(
        uint256 _eventId,
        string memory _location,
        string memory _metadata
    ) public onlyRegistered {
        Event memory eventDetails = events[_eventId];
        
        require(eventDetails.active, "Event is not active");
        require(block.timestamp >= eventDetails.startTime && block.timestamp <= eventDetails.endTime, 
                "Attendance can only be marked during the event");
        
        // Check if attendance has already been marked
        require(attendanceRecords[_eventId][msg.sender].timestamp == 0, "Attendance already marked");
        
        // Record the attendance
        attendanceRecords[_eventId][msg.sender] = Attendance({
            timestamp: block.timestamp,
            location: _location,
            metadata: _metadata
        });
        
        emit AttendanceMarked(_eventId, msg.sender, block.timestamp);
    }
    
    /**
     * @dev Register a new participant
     * @param _participant Address of the participant to register
     */
    function registerParticipant(address _participant) public onlyOwner {
        require(_participant != address(0), "Invalid participant address");
        require(!registeredParticipants[_participant], "Participant already registered");
        
        registeredParticipants[_participant] = true;
        
        emit ParticipantRegistered(_participant);
    }
}
