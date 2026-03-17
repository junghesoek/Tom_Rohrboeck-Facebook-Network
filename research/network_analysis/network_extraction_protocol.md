# Network Extraction Protocol - Tom Rohrböck Facebook Network

## Automated Network Discovery System

### Phase 1: Direct Connection Extraction
```javascript
// Complete friends list extraction
async function extractCompleteFriendsList() {
  const friends = [];
  let scrollCount = 0;
  const maxScrolls = 1000;
  
  // Navigate to friends section
  await browser.navigate("https://www.facebook.com/tom.rohrbock.1/friends");
  await browser.wait(3000);
  
  // Scroll through entire friends list
  while (scrollCount < maxScrolls) {
    // Extract visible friends
    const visibleFriends = await browser.extractElements('[data-testid="friend_list_item"]');
    
    for (let friend of visibleFriends) {
      const friendData = {
        name: await browser.getText(friend.nameSelector),
        profileUrl: await browser.getAttribute(friend.profileSelector, 'href'),
        mutualFriends: await browser.getText(friend.mutualSelector),
        profileImage: await browser.getAttribute(friend.imageSelector, 'src'),
        connectionType: await analyzeConnectionType(friend),
        lastInteraction: await browser.getText(friend.lastInteractionSelector),
        verifiedProfile: await browser.exists(friend.verifiedBadgeSelector)
      };
      
      friends.push(friendData);
      
      // Extract friend's profile details
      await extractFriendProfileDetails(friendData);
    }
    
    // Scroll down for more friends
    await browser.scrollDown();
    await browser.wait(2000);
    
    // Check if we've reached the end
    const newFriends = await browser.extractElements('[data-testid="friend_list_item"]');
    if (newFriends.length === 0) break;
    
    scrollCount++;
  }
  
  return friends;
}
```

### Phase 2: Mutual Friend Analysis
```javascript
// Analyze mutual connections between friends
async function analyzeMutualConnections(friendsList) {
  const mutualAnalysis = {};
  
  for (let friend of friendsList) {
    // Navigate to friend's profile
    await browser.navigate(friend.profileUrl);
    await browser.wait(2000);
    
    // Extract mutual friends
    const mutualFriends = await extractMutualFriends();
    
    mutualAnalysis[friend.name] = {
      mutualWithTom: mutualFriends.length,
      mutualInNetwork: mutualFriends.filter(mf => 
        friendsList.some(f => f.name === mf.name)
      ),
      networkClusters: identifyNetworkClusters(mutualFriends)
    };
    
    // Extract friend's friend connections
    const friendConnections = await extractFriendConnections(mutualFriends);
    mutualAnalysis[friend.name].connections = friendConnections;
  }
  
  return mutualAnalysis;
}
```

### Phase 3: Political Network Identification
```javascript
// Identify and categorize political connections
async function identifyPoliticalNetwork(friendsList) {
  const politicalNetwork = {
    afd: [],
    cdu_csu: [],
    fdp: [],
    grüne: [],
    spd: [],
    other: [],
    unknown: []
  };
  
  for (let friend of friendsList) {
    const politicalAffiliation = await analyzePoliticalAffiliation(friend);
    
    if (politicalAffiliation.party) {
      politicalNetwork[politicalAffiliation.party].push({
        ...friend,
        role: politicalAffiliation.role,
        position: politicalAffiliation.position,
        influence: politicalAffiliation.influence,
        verified: politicalAffiliation.verified
      });
    } else {
      politicalNetwork.unknown.push(friend);
    }
  }
  
  return politicalNetwork;
}

async function analyzePoliticalAffiliation(friend) {
  const profile = await browser.extractProfileContent(friend.profileUrl);
  
  // Check for political indicators
  const politicalIndicators = {
    partyAffiliations: extractPartyMentions(profile.about),
    politicalPages: extractLikedPages(profile.pages, ['political', 'party']),
    politicalPosts: analyzePoliticalContent(profile.posts),
    politicalGroups: extractGroupMemberships(profile.groups, ['political']),
    politicalWork: analyzeWorkHistory(profile.work),
    politicalEducation: analyzeEducation(profile.education)
  };
  
  return categorizePoliticalAffiliation(politicalIndicators);
}
```

### Phase 4: Business Network Mapping
```javascript
// Map business and professional connections
async function mapBusinessNetwork(friendsList) {
  const businessNetwork = {
    corporateConnections: [],
    investmentPartners: [],
    consultingClients: [],
    boardMemberships: [],
    industryAssociations: [],
    professionalServices: []
  };
  
  for (let friend of friendsList) {
    const businessProfile = await analyzeBusinessProfile(friend);
    
    if (businessProfile.hasBusinessConnection) {
      const businessCategory = categorizeBusinessConnection(businessProfile);
      
      businessNetwork[businessCategory].push({
        ...friend,
        company: businessProfile.company,
        position: businessProfile.position,
        industry: businessProfile.industry,
        relationshipType: businessProfile.relationshipType,
        businessEvidence: businessProfile.evidence
      });
    }
  }
  
  return businessNetwork;
}
```

### Phase 5: Geographic Network Analysis
```javascript
// Analyze geographic distribution of network
async function analyzeGeographicNetwork(friendsList) {
  const geographicNetwork = {
    seligenstadt: [],
    hessen: [],
    bavaria: [],
    germany: [],
    international: [],
    unknown: []
  };
  
  for (let friend of friendsList) {
    const location = await extractLocationData(friend);
    
    const geographicCategory = categorizeLocation(location);
    
    geographicNetwork[geographicCategory].push({
      ...friend,
      location: location,
      distance: calculateDistance(location),
      regionalInfluence: assessRegionalInfluence(friend, location)
    });
  }
  
  return geographicNetwork;
}
```

### Phase 6: Network Influence Analysis
```javascript
// Analyze influence patterns and chains
async function analyzeNetworkInfluence(networkData) {
  const influenceAnalysis = {
    centralNodes: identifyInfluentialNodes(networkData),
    bridgePersons: identifyBridgePersons(networkData),
    influenceChains: mapInfluenceChains(networkData),
    networkClusters: identifyNetworkClusters(networkData),
    influenceMetrics: calculateInfluenceMetrics(networkData)
  };
  
  return influenceAnalysis;
}

function identifyInfluentialNodes(networkData) {
  // Calculate influence scores based on:
  // - Connection count
  // - Political positions
  // - Business influence
  // - Geographic centrality
  // - Cross-network connections
  
  const influenceScores = {};
  
  for (let person of networkData.allConnections) {
    let score = 0;
    
    // Connection influence
    score += person.connectionCount * 0.2;
    
    // Political influence
    if (person.politicalRole) {
      score += getPoliticalInfluenceScore(person.politicalRole) * 0.3;
    }
    
    // Business influence
    if (person.businessInfluence) {
      score += getBusinessInfluenceScore(person.businessInfluence) * 0.25;
    }
    
    // Geographic influence
    score += getGeographicInfluenceScore(person.location) * 0.15;
    
    // Network bridge influence
    score += getBridgeInfluenceScore(person) * 0.1;
    
    influenceScores[person.name] = score;
  }
  
  return sortByInfluence(influenceScores);
}
```

## Evidence Collection System

### URL Collection Protocol
```javascript
// Comprehensive URL collection
async function collectAllUrls(networkData) {
  const urls = {
    profileUrls: [],
    photoUrls: [],
    postUrls: [],
    groupUrls: [],
    pageUrls: [],
    eventUrls: []
  };
  
  for (let connection of networkData.allConnections) {
    // Collect profile URL
    urls.profileUrls.push({
      name: connection.name,
      url: connection.profileUrl,
      type: 'profile',
      accessible: await testUrlAccessibility(connection.profileUrl),
      lastChecked: new Date().toISOString()
    });
    
    // Collect additional URLs from profile
    const additionalUrls = await extractAdditionalUrls(connection.profileUrl);
    urls = mergeUrls(urls, additionalUrls);
  }
  
  return urls;
}
```

### Screenshot Documentation
```javascript
// Systematic screenshot collection
async function collectNetworkScreenshots(networkData) {
  const screenshots = {
    profileOverviews: [],
    friendLists: [],
    networkConnections: [],
    politicalConnections: [],
    businessConnections: [],
    geographicMappings: []
  };
  
  for (let connection of networkData.priorityConnections) {
    // Navigate to profile
    await browser.navigate(connection.profileUrl);
    await browser.wait(3000);
    
    // Take comprehensive screenshots
    const profileScreenshot = await browser.takeScreenshot('full_page');
    screenshots.profileOverviews.push({
      name: connection.name,
      filename: `profile_${connection.name.replace(/\s+/g, '_')}.png`,
      timestamp: new Date().toISOString(),
      url: connection.profileUrl
    });
    
    // Extract network visualization data
    const networkData = await extractNetworkVisualization(connection);
    screenshots.networkConnections.push(networkData);
  }
  
  return screenshots;
}
```

## Data Integration and Analysis

### Network Graph Generation
```javascript
// Generate comprehensive network graph
function generateNetworkGraph(networkData) {
  const graph = {
    nodes: [],
    edges: [],
    clusters: [],
    metadata: {
      totalNodes: networkData.allConnections.length,
      totalEdges: 0,
      clusterCount: 0,
      density: 0,
      centralNodes: []
    }
  };
  
  // Add central node (Tom Rohrböck)
  graph.nodes.push({
    id: 'tom_rohrboeck',
    label: 'Tom Rohrböck',
    type: 'central',
    size: 50,
    color: '#ff0000'
  });
  
  // Add all connections as nodes
  for (let connection of networkData.allConnections) {
    graph.nodes.push({
      id: connection.id,
      label: connection.name,
      type: connection.type,
      size: calculateNodeSize(connection),
      color: getNodeColor(connection),
      metadata: connection
    });
    
    // Add edge to central node
    graph.edges.push({
      from: 'tom_rohrboeck',
      to: connection.id,
      strength: connection.connectionStrength,
      type: connection.relationshipType,
      weight: calculateEdgeWeight(connection)
    });
  }
  
  // Add mutual friend edges
  for (let mutualConnection of networkData.mutualConnections) {
    graph.edges.push({
      from: mutualConnection.from,
      to: mutualConnection.to,
      strength: mutualConnection.strength,
      type: 'mutual_friend',
      weight: calculateEdgeWeight(mutualConnection)
    });
  }
  
  return graph;
}
```

---
**Ready for execution once browser connection is established**  
**Will systematically map entire Facebook network with evidence**
