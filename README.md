# building_network_tech_refresh
**Network Tech Refresh Tasks for Buildings**


<img src="https://github.com/feralpacket/building_network_tech_refresh/blob/main/github_idf_cabinet.jfif" width=50% height=50%>


When doing a tech refresh of the network for an entire building, there is quite a bit that needs to be considered, tracked, and completed.  I've created a list of the tasks that I do when doing a tech refresh.  

We do the tech refreshes for vendor support reasons.  Software upgrades for new features and security fixes are part of it.  Mainly we want to make sure the hardware is supported and will be replaced when it fails or stops working properly.

**Building Network Design**

Quick description of how the network in each building is laid out so the tasks I list below make more sense.  The building distribution switches are called MDFs ( Main Distribution Frame ).  The access layer switches are called IDFs ( Intermediate Distribution Frame ).  The IDFs are numbered starting with 1.  The number of IDF depend on the size of the building and range from 6 IDFs to well over 50.  Some of the limiting factors with the network design is the port density on the MDFs, the location of fiber coming into the building. and how the fiber and cable trays are run in the building.

For the larger warehouses, we tend to have a non-standard design.  We will have 2 MDFs, 2 - Cisco 9500 virtual Stackwise pairs.  Each pair has multiple, separate fiber uplinks to the campus distribution switches.  The 2 MDF switches are interconnected with trunk links.  The trunk links between the MDFs are needed to allow for the VLAN and IP address schema we use in each building.  The IDFs are split into multiple zones.  Typically, half of the IDFs are connected to 1 MDF, the other half are connected to the 2nd MDF.  Sometimes we are forced to use 2 or more MDFs for large buildings that were built in phases.  Each phase ends up with it's own fiber loop.  Asking management to rerun fiber through out a large building is a challenge at best.  Even for new buildings that only have 1 fiber loop, we sometimes have port density issues at the MDF.  While getting a chassis based switch such as the Cisco 9600 would solve that problem, management balks at the price.  At the end of the day, I have to make due with what management buys.  Smaller buildings will only have 1 MDF.

**VLANs and IP Address Schema**

We standardize the IP addressing schema to ease with configuration and troubleshooting not only with the Network Team and Help Desk, but also with the employees and vendors.  The IP addressing for the building is 10.[building number].0.0 /16.  Each IDF get their own VLAN and /24 subnet.  We call these IDF VLANs.  IDF 1 in Building 1 will be configured for VLAN 101, with the IP address subnet 10.1.101.0 /24.  IDF 55 in Building 7 will be configured for VLAN 155, with the IP address subnet 10.7.155.0 /24.  Management IPs follow along with the IDF VLAN number.  IDF 1 in Building 2 will have a management IP address of 10.2.2.101.  IDF 2 in Building 2 will have a management IP address of 10.2.2.102.  The number of Voice VLANs vary depending on the size of the building and number of IDFs.  IDF 1 in Building 3 will have a Voice VLAN of 201 with the IP address subnet 10.3.201.0 /24.  The same Voice VLAN will be configured on multiple IDFs.  We don't have more than 10 Voice VLANs in our largest buildings.  We have what we call non-IDF VLANS.  The Printer VLAN will be VLAN 10 in all of the buildings.  For Building 1, the Printer VLAN will be VLAN 10 with the IP address subnet 10.1.10.0/24.  Time clocks will use VLAN 11 in all of the buildings.  Campus security cameras, badge readers, mag locks, and other devices are spread across their own VLANs.  Vendor equipment will get their own VLAN and subnet for each building.  They tend to be one offs or limited to a small number of buildings.  The PLCs that run the conveyors systems will get assigned a VLAN.  That VLAN number will then be used for any building that has the same conveyor system from the same vendor.  In the MDFs, there is and IDF MDF switch.  This is needed to provide copper ports for equipment such as the UPS, security cameras, badge readers, IP phones, and any other equipment in the MDF that needs network connectivity.  The IDF MDF switch uses VLAN 100 and the appropriate IP address subnet.  If there is a second MDF IDF switch, it will use VLAN 99.

When configuring the new switches, I configure all of the interfaces for the IDF VLAN.  This makes initial configuration easier to script and automate.  Configuring the interfaces using the non-IDF VLANs before hand has the problem of not catching interface VLAN changes between when the new switch was configured and when it replaces the old switch.  This could only be a few days.  Sometimes there is a month gap.  I've found getting the rest of the Network Team to track interface VLAN changes is a challenge.  I configure the interfaces using non-IDF VLANs after the cut-over or replacement of the IDF switch.  Yes, this causes more work during the cut-over.  In the end, I've found it's easier and causes fewer problems with endpoints.

**Tech Refresh Goals**

One of the main goals of the tech refresh is to minimize network down time during the cut-over or replacement of the switches.  But tech refreshes rarely occur without something going wrong.  Fortunately, we have a Hardware Team that does a bulk of the physical labor.  Sometimes, the cut-over is done over a weekend.  Usually, the cut-overs are spread over a week or two.  A small number of IDFs are replaced in the afternoons or early evenings.  The morning after a cut-over, someone from the Hardware Team shows up at 5 AM and checks with the employees and vendors to see if there are any problems.  Replacing only a few IDFs at a time makes this portion of the tech refresh easier for the Hardware Team and the employees.  If all of the switches were replaced over a weekend, trying to check with all of the users spread across a large building at the same time tends to cause a lot of frustration with everyone involved.  The problem with spreading to cut-overs across multiple days is the network is in a partial transition state where some of the IDFs are connected to the new MDFs while some of the IDFs are still connected to the old MDFs.  This needs to be white boarded and checked a few times to make sure we don't run into spanning-tree diameter problems.  Bandwidth can also be a temporary problem if the old IDFs do not have enough 10G interfaces to support temporary links between the old MDFs and new MDFs.  While I could probably route the cut-over IDFs out the new MDFs during this transition phase, the non-IDF VLANs can potentially be a problem.  I've found it's easier to just cut-over all of the routing and switching at once.  

If new IDFs are added as part of the tech refresh, then I usually end up having to renumber most of the IDFs.  I don't want to have to live with IDF 1, IDF 2, IDF 45, IDF 3, IDF 46, IDF 4, etc.  This means temporary management IP addresses need to be used with the new, replacement switches.  Old IDF 7 might be renumberd to new IDF 9, but old IDF 9 may still be online at some point during the cut-overs.  This causes additional work and configuration later with device authentication and network monitor when the temporary management IP addresses are replaced with the permanent management IP addresses.  Renumbering IDFs also means that more documentation is needed and more communication with everyone involved is required to try to avoid too much confusion.  As painful as it can be during the tech refresh, the end result of a consistent IDF numbered layout is preferred.

**Task List**

The task list below is only for the wired, switched network.  The task list for the wireless portion of the tech refresh is not included.  I have tried to be as complete as possible, mainly to help train our junior Network Team members.  But I may of missed or left out a few steps.  The commands I've included are Cisco specific.

If you are crazy enough to use some of the tasks listed below, make the appropriate changes to match your network, environment, management, and employees.  Some of the steps below are actually automated.

**The progress tracker I create and use:**

![Progress Tracker](https://github.com/feralpacket/building_network_tech_refresh/blob/main/github_progress_tracker.png)

![Progress Tracker](https://github.com/feralpacket/building_network_tech_refresh/blob/main/github_progress_tracker_2.png)

**The PoE usage review:**

![PoE Usage](https://github.com/feralpacket/building_network_tech_refresh/blob/main/github_poe_usage.png)

![PoE Usage](https://github.com/feralpacket/building_network_tech_refresh/blob/main/github_poe_usage_2.png)

**The MDF cutsheet:**

![MDF Cutsheet](https://github.com/feralpacket/building_network_tech_refresh/blob/main/github_cutsheet.png)


```
! Building tech refresh tasks
  - Create new progress tracker
    - Important for large buildings that will have a lot of equipment replaced
    - Helps track projects that are spread over a long period of time
      - The global supply chain problems have caused 6 month or more lead times for network equipment 
        to be delivered


  - Create a new projects folder


  - Identify switches that need to be replaced
    - Create a list
    - Identify switch stacks and number of switches in the stack
    - Identify any "extra" network equipment managed by the Network Team in the building
      - Equipment staging switches
      - Lab equipment
      - Printer cart switches


  - Identify any changes to the building
    - Check with management to see if there are any changes
    - Check with the Hardware Team to see if there are any changes
    - Check with campus security to see if there are any changes
    - Check with the offices and sections in the building to see if there are any changes
	  - Management usually does this
    - Building expansion?
    - Existing fiber replaced?
    - Any new IDFs?
    - Any IDFs being removed or moved?
    - Any increase in the number of APs?
    - Any increase in the number of IP phones?
    - Any increase in the number of security cameras?


  - Current switch review
    - This is added to the progress tracker
    - Switch stacks
      - sh switch
    - Current port density review
      - sh int status | count connected
    - Current PoE usage
      - sh power inline | ex Gi
    - Current number of APs connected
      - sh cdp nei | count AIR


  - Tour the building
    - Visit
      - MDF locations
      - IDF locations
      - Pick modules
      - Any special vendor equipment
	- Conveyor systems
	- Cranes
	- Network connected industrial equipment
	- Large Heidelberg printers
      - The mahogany desk user suite ( do this after hours )
      - Any special offices or sections or labs
        - Do they have any unique requirements?
        - Do they have any unique network equipment?
      - Anything that will need special attention
    - Identify any problems
      - How many fiber loops?
      - MDF locations
        - Check fiber uplink availability to the campus distribution switches
          - To avoid last minute changes when the Hardware Team realizes they are short on fiber
          - and have to patch through several other buildings to reach the campus distribution switches
          - Do a fiber link loss analysis if this is the case
        - Check for available power
        - Check the PDUs
        - Check the availability of convenience outlets
          - What you plug your laptop into during the cut-overs
        - Check the UPS
          - New UPS are typically ordered for the MDFs
          - If old UPS are kept
            - When was the last time the batteries were replaced?
      - IDF cabinets / racks
        - Check space availability
          - Identify full cabinets
            - Typically a problem with large switch stacks and associated patch panels
        - Weight limitations
        - Check the datasheet for the cabinet
        - Fiber uplink availability
        - Do the mounting rails in the cabinet need to be moved for the new switches?
          - Sometimes not easy to do
          - Needs to be done before the cut-overs are scheduled
        - Check for available power
        - Check the PDUs
          - If the IDF is mounted in a rack
        - Check the availability of convenience outlets
          - What you plug your laptop into if there is a problem
        - UPS
          - Existing UPS are typically kept for the IDFs
          - When was the last time the batteries were replaced?
        - Check for potential cooling problems
          - Can be an issue with full cabinets
          - Can be an issue with warehouses with low, metal roofs during the summer
            - The IDF cabinets mounted 15 feet above the ground can be in the hot air close to the ceiling
    - Do the MDFs have chairs or a place to use as a desk?
      - If not, you will have to grab a couple of folding chairs from a break room when needed
    - Where are the boom lifts and scissor lifts located?
      - The Hardware Team will be using them during the cut-over
      - Still a good idea to know where any unused equipment is located
    - Where are the tornado shelters?
      - The MDF rooms tend to be the best location
        - But may not be near the exterior of the building ( it would take a while for rescuers to dig you out )
        - Does not have a water source
      - The restrooms co-located in the same structure as the MDF would be a better location
      - The buildings tend to be empty during cut-overs, meaning you lose the ability of following the 
        other employees to the tornado shelter
    - Where are the restrooms?
    - Where are the break rooms?


  - Replacement network equipment selection
    - Review current switch models available from Cisco
    - Review current optics available from Cisco
    - Consider
      - MDF
	- Port density ( number of IDFs )
	- Routing features
	- Security features
	- SSD / VM support and features
      - IDF
	- Port density
	- PoE usage
	- Secondary power supplies
	- Network modules ( for uplinks )
	- Security features
    - Check switch and IDF cabinet dimensions
      - The C9300-48P barely fits in the IDF cabinets we use in the buildings


  - Campus / building network design
    - Review Cisco's latest SRND / CVD / best practices for campus networks
    - Look for changes, recommendations, new technologies compared to what is being used in the building
    - Some technologies will require additional network equipment or services to be install
      - And can require expensive testing in a lab environment to minimize downtime during a tech refresh
      - Cisco SD-Access, for example
    - Consider any changes being made to the building
      - Fiber
      - IDFs
        - New IDFs are commonly added to resolve copper cable distance problems with APs or security cameras
        - Will there be a large number of new devices that will require the port density to increase?
          - Management should provide this information
          - Or tell you who should provide this information
          - Or just ask the Hardware Team
    - Check with the Hardware Team on the fiber loops in the building
      - Some of the buildings were built in phases
        -This results in multiple, separate fiber loops
      - Will significantly impact the network design
        - Example building built in phases
          - Existing fiber loops means it has 2 MDFs
	      - 1 MDF for 2/3rds of the phase 1 part of the building
          - 1 MDF for 1/3rd of the phase 2 part of the building
          - Also means there will need to be links between the 2 MDFs
        - Example building, 2 million square foot warehouse
          - Built with 3 fiber loops because of it's size
          - Means there are 3 MDFs
          - Each MDF covers 1/3 of the building
          - 3rd MDF only connects to the first 2 MDFs
          - No, this design is not covered in any Cisco Design Guide
    - Consider temporary links needed before and during the cut-over


  - Consider new technologies
    - SD-Access
    - SGT
    - may influence the switch models selected


  - Identify unique equipment connected to the IDF switches not managed by the Network Team
    - Unmanaged switches or hubs
    - Voice Gateways
    - UPS
    - Panic buttons
    - IT Security honeypots
    - Network equipment for
      - Labs
      - Special offices or sections
    - Vendor equipment or managed switches
    - Unique protocols not used in other buildings
      - Multicast for vendor equipment


  - Network diagram review
    - Update existing network diagram to reflect current state
    - Create new network diagram to reflect changes after the tech refresh


  - IDF layout review
    - Update existing IDF layout to reflect current state
    - Rename IDFs if necessary
    - Create new IDF layout to reflect changes after the tech refresh


  - Documentation review
    - Make sure doku is up to date
      - Building VLAN list
      - Campus distribution switch information
      - BGP / routing information
      - Anything unique about the building
      - Unmanaged switches are documented
      - Voice gateways and panic buttons are documented


  - IP Address Management review
    - Verify
      - Current loopbacks are listed
   	  - Current uplinks are listed 
      - VLANs are listed


  - DHCP review
    - Static reservations
      - Check to see if they are active
      - Check computer names against current AD computer objects
      - Ask management, the Hardware Team, system administrators, help desk if there are any that 
        should be deleted
      - Delete unused reservations if there are no objections
    - Current DHCP pool usage
    - Identify any special, one off DHCP options
      - Staging rooms commonly uses DHCP options for ZTP or Auto-Install or PXE Boot


  - VLAN review
    - Identify any special, one off VLANs not present in all of the campus buildings?
    - Identify anything that needs to be added ( new IDFs ) or deleted before or after the cut-over?


  - Create the Bill of Materials ( BOM )
    - Include
      - MDF
     - Switches
       - Power supplies
       - Network modules
       - SSD
       - Uplink optics
       - IDF uplink optics
       - Virtual stackwise optics
       - Dual active detect optics
       - Links between MDFs if there are 2 or more MDFs
    - IDF
       - Switches
       - Power supplies
       - Secondary power supplies
       - Network modules
       - Stack cables
    - Other
       - Any spares need to be added?
       - Any temporary optics needed?
       - Old MDF to new MDF temporary links
    - Future PoE usage based on new switch and power supply model
      - Add to the progress tracker
      - 2960x usually have 740 watts available for PoE
      - The C9300-48P with 715 WAC power supply will have 472 watts available for PoE
      - Consider any increases in APs, IP phones, or security cameras
      - Add secondary power supplies as necessary


  - Review the BOM
    - Review the BOM
    - Review it, again
    - Remember
      - Optics for temporary links
        - Both the old MDF and new MDF switches will be online at the same time
          - They will need a couple of trunks links for the cut-over process
          - They will need uplinks to the campus distribution switches at the same time
      - Stack cables, switch stacks of 3 or more will need longer stack cables
      - PoE usage
        - Order secondary power supplies if necessary
      - Licensing
        - Shouldn't be an issue since we have an EA
    - Review the BOM, again
    - Ask the rest of the Network Team review the BOM ( peer review )
    - Review the BOM with management
    - Whiteboard the tech refresh
    - Seriously, what did you forget?
    - With the global supply chain problems, if you forgot to add something to the BOM, it might 
      take a year to arrive if you have to order something later.


  - Cut the Purchase Order ( PO )
    - And... wait


  - Create the cut-sheets
    - Will be used by the Hardware Team
      - They will label the non-IDF VLAN interfaces and cables ahead of time
	  - Speeds up cut-overs
	  - Results in less problems with endpoints being in the wrong VLAN
    - MDFs
      - List current connections
        - List switch at other end of link
        - List cabling
          - SM
          - MM
            - OM2
            - OM3
            - OM4
          - Copper
         - List connection speed
         - List port-channels
      - List new connections
      - List temporary connections
        - Links between old and new MDF switches
          - Keep in mind the interfaces available on the old MDFs
          - Should be a 10G link if the IDFs are not cut-over all at once over a weekend
          - Check availability of 10G interfaces on the old MDFs
            - For one building, had to repurpose one of the uplinks to the campus distribution switches
            - The MDF switches were out of available 10G interfaces
        - Management interface for the new MDF switches, used until the cut-over is complete
    - IDFs
      - List current uplinks
      - List future uplinks
      - List interfaces with APs
      - List interfaces configured for non-IDF VLANs
      - The Hardware Team will label the cables for the non-IDF VLAN interfaces before the cut-over


  - Temporary IDF IP addresses
    - If need or being used
      - Create a table mapping
        - Old IDF name
        - Old management IP address
        - New IDF name
        - Temporary management IP address
        - Permanent management IP address
    - Inform the Network Team
      - So they can login during the cut-over process if necessary
    - Inform the Telecom Team
      - THIS IS REALLY IMPORTANT
      - So they can update Cisco Emergency Responder ( CER ) information during the cut-over process
      - So 911 and the fire department show up to the proper door and not the opposite side of a 
        2 million square foot warehouse


  - Current switch configuration
    - Make an offline back up of the current configuration of the switches
    - Include
      - ARP table
        - Information is helpful when some device with a stack IP address stops working after 
	  the cut-over and you need to figure out what it is and where it used to be connected
      - MAC address table
      - CDP neighbors
      - LLDP neighbors
      - Routing neighbors and routing table, for MDF switches
    - Use the show run script


  - Review current switch logging messages
    - sh logging
    - Check the syslog server
    - Identify current problems
      - Optic low power alarms
      - Interfaces that are flapping
        - up / down
        - PoE applied / removed
      - Dot1x failures
      - One off, unique, strange, rare logging messages
    - Contact to see if they have any known network problems
      - Campus security
      - Offices
      - Vendor equipment in the building
      - Check the sections in the building
        - Example, the printer department with their big Heidelberg printers
		- Example, the automated cranes and robots
    - Fix the problems before the cut-over
      - It's not uncommon for security, offices, sections, vendors, etc to complain about issues 
        after the cut-over that actually existed before the cut-over
    - Create incidents for the Hardware Team to troubleshoot any cabling issues


  - Identify the recommended IOS version to use on the new switches
    - Review release notes
    - Identify new features that can be used
      - Test in the lab
    - Identify resolved issues
    - Identify open issues that affect features being used in the building


  - IP Address Management
    - Reserve loopback IP addresses for the new MDF switches
      - To be used as the router-id for the routing protocol
    - Reserve uplink subnets for the new MDF switches
    - Reserve IP addresses for any new VLANs


  - Create new MDF switch configuration
    - Review switch baseline
      - Add features or configurations if necessary
    - Configure virtual stackwise
    - Configure routing
      - Configuring BGP
        - Do not activate the neighbor
        - Use a temporary route-map to set MED so routing prefers the existing MDFs
    - Configure VLANs
    - Configure SVIs
      - Leave shutdown until needed during IDF cut-overs
      - Except for VLAN 1
      - Use temporary IP address for VLAN 1
        - .2 if HSRP is not being used
        - .4 and .5 if HSRP is being used
        - .2 and .3 is normally used with HSRP
    - Configure VTP
      - We use VTP version 3
      - The VTP primary will remain on one of the existing MDF switches until routing and switch 
        is cut-over
    - Configure temporary trunk links between old and new MDFs
    - Configure HSRP
      - If there are 2 MDF switches
    - Configure spanning-tree
      - Spanning-tree VLAN priority set so the existing MDFs retain the root bridge
    - Temporary configuration
      - Management interface
        - connected to an IDFMDF switch
        -  Configure a temporary IP address
      - Configure source-interface gi0/0 services
        - RADIUS
        - TACACS
        - logging
        - NTP
        - SSH
        - HTTP server for ISE
        - HTTPS server for ISE
    - Unique protocols not used in other buildings
      - Typically for vendor equipment
        - Layer 2 multicast being used, for example


  - Create new IDF switch configurations
    - Review switch baseline
      - Add features or configurations if necessary
    - Identify new features
      - Test them in the lab
      - Test them again
      - Ask the Network Team to test them ( peer review )
    - Switch stacks
      - Don't forget about these
    - VLAN 1
      - If IDF is being renumber / renamed and the management IP address / IDF VLAN will change
        - Use temporary IP address
          - Instead of 10.x.x.101, use 10.x.x.201
          - Verify there will be no IP conflict with existing network devices in the building
            - Double check the IP addresses used by the voice gateways
    - Access port interfaces are initially configured with the IDF VLAN
      - Makes it easier to configure a large number of switches
      - Interfaces using non-IDF VLANs will be configured as part of the IDF cut-over
    - Unique protocols not used in other buildings
      - Typically for vendor equipment
        - Layer 2 multicast being used, for example


  - Create campus distribution switch configuration changes
    - Temporary uplinks for new distribution switches
    - Routing changes
      - Advertise new uplink subnets
      - New neighbors


  - Create core switch configuration changes
    - Routing changes
      - Redistribute new uplink subnets
      - Redistribute new loopback IP addresses
      - Can be configured before new distribution switches are installed


  - Create existing MDF configuration changes
    - Temporary links between new MDFs to old MDFs
    - Check to see if the new MDFs are using new VLANs ( needed if IDFs are being added to the building )
      - If so, configure new VLANs and SVIs
    - Routing change
      - Advertise any new subnets or SVIs


  - ISE configuration changes ( network device authentication )
    - Add network device objects for any new equipment
    - Add network device objects for temporary IP addresses
      - Keep in mind, the new MDFs will be using temporary IP addresses
      - Typically, the IP addresses configured on the management interface


  - DHCP configuration changes
    - Configure any new DHCP scopes
      - If there are new IDFs
      - Don't forget to configure failover for the new DHCP scopes


  - Switches are delivered
    - The fun begins


  - Configure the new switches
    - Create ticket for inventory management to pull switches, stack cables, NIMs, secondary 
      power supplies
    - Start with one switch
      - Install power supply
      - Install secondary power supply
      - Install NIM
      - Power up
      - Review IOS version
        - sh version
        - Check to see if a new IOS version was recently released
          - Review release notes, again
        - ZTP will be need if it doesn't match what was selected to be used
      - Check installed devices
        - sh inventory
          - NIM present?
          - Secondary power supply present?
          - SSD present?
      - Check power supply and fans
        - sh environment all
        - This command tends to change from time to time
      - Check logging messages
        - look for anything that is not normal
    - ZTP
      - If needed
      - Upload new IOS to FTP / TFTP server
      - Change the ZTP script, if necessary
      - Test on the first switch
    - ZTP done
      - For switch stacks
        - Renumber switches
        - Configure switch priority
        - Unplug power
        - Connect stack cables
        - Plug in power
        - Verify switch stack
          - sh switch
      - Review IOS version
        - sh version
      - Check installed devices
        - sh inventory
          - NIM present?
          - Secondary power supply present?
          - SSD present?
      - Check power supply and fans
        - sh environment all
        - This command tends to change from time to time
      - Check logging messages
        - Look for anything that is not normal
      - Apply configuration
      - Save the configuration
        - copy running-config startup-config ( wr mem )
      - Reload
      - Verify
        - Can login with local account
        - VLAN 1 is not shutdown
          - sh ip int bri | ex ss
        - VTP
          - sh vtp status
        - Default route
          - sh ip route
          - sh run | in default
        - Interfaces
          - sh int status
          - sh run int gi1/0/1
        - Interfaces on switch stacks
          - sh run int gi2/0/1
          - sh run int gi3/0/1
        - Trunk links
          - sh run int te1/1/1
          - sh run int te2/1/1
        - Port-channels
          - sh etherc summary
        - Logging messages
          - sh logging
          - Look for anything that is not normal


  - Prep for the Hardware Team
    - Create label for switch
      - Include switch number for switch stacks
      - Building1-IDF1, Switch 1
    - Place switch back in box
      - Remember to include the stack cables
    - Write IDF number on box
      - Buidling1-IDF1, Switch 1
    - Inform the Hardware Team when all of the switches are ready


  - Configure existing IDFMDF switches
    - Configure interface for the new MDF management interface
    - VLAN 1


  - MDF install
    - The Hardware Team likes to do this a month before the IDFs are installed and cut-over
      - for testing and "burn in"
      - *shrug*
    - Rack
    - Power
    - Connect management interface to IDFMDF switch
    - Verify
      - remote access
      - NTP
        - Usually takes 5 - 10 minutes to synchronize
        - sh ntp ass
        - sh ntp ass detail
        - sh ntp status
      - IP reachability
        - ping campus DNS server
          - ping 8.8.8.8
        - ping colo DNS server
          - ping 8.8.4.4
      - Check inventory
        - Did something get damaged or stop working?
          - sh inventory
      - Check power supply and fans
        - Did something get damaged or stop working?
          - sh environment all
    - Take a picture for documentation and for Twitter
    - Configure campus distribution switches
      - New uplinks
      - Routing changes
    - Connect uplinks
      - Sometimes, this is done a week or two after the MDF switches are racked
      - Hence the need for the management interface to be configured
      - Verify
        - Link up / up
        - IP reachability on the link
        - Routing neighbor is up
        - Routing information is being exchanged
        - Router-id for new MDFs is being advertised
        - Existing VLANs and SVIs are still being advertise by the old IDFs
    - Connect old MDF to new MDF links
      - Verify
        - Trunk links are up
          - sh int trunk
        - New MDFs received VLAN configuration via VTP
          - sh vlan bri
          - sh vtp status
          - Sometimes, this will not happen
          - On the old MDF switch that is the VTP primary, run the global exec command "vtp primary"
        - Spanning-tree
		  - Old MDFs are still the root bridge
		  - sh spanning-tree | include ^VLAN|root


  - IDF switch install
    - Install before cut-over if possible
    - Sometimes this will be done over a weekend
      - Tt's a long weekend, but there tend to be less network problems then during partial 
        cut-overs spread across a week or two
    - Sometimes this will be done a handful of IDFs at a time, spread over a week or two
      - The Hardware Team prefers it
        - No long weekend
        - Only have to be present the next morning in a smaller part of the building to verify 
	  users are not having problems
          - Instead of having to check an entire 2 million square foot building
      - Thank you Hardware Team for assuming I'll be happy and willing to work late 
        nights for 2 weeks
    - Sometimes
      - The new IDF switches can be racked before the cut-over
      - Speeds up the cut-over
    - Sometimes
      - The IDF cabinet is full, the existing IDF switch will need to be unracked before the 
        news ones are racked


  - Create IDF after cut-over configuration changes
    - Since all of the interfaces are the IDF switches are initially configured for the IDF VLAN
    - The interfaces that use non-IDF VLANs will need to be configured after the cut-over
    - Use the copy of the existing switch configuration created earlier
    - Check the interfaces currently being used
    - There usually are differences
      - Interface VLAN changes are common
      - Some equipment, printers, are only power up when they are used
      - Some equipment may be down for weeks before the office or section complains
	- For notconnect interfaces
	  - Using the tech refresh as an opportunity to clean up unused interfaces
	  - If the interface is notconnect with the configuration was backed up previously and is 
	    notconnect when the after cut-over configuration is created
	  - Been leaving those interfaces configured with the IDF VLAN
	  - Not uncommon for 4 interfaces to be configured for time clocks, but only 2 time clocks 
	    are being used
	  - Not uncommon for new devices to be connected to the network, but the interfaces used 
	    by the old devices were not cleaned up
	  - May be a problem with printers, some users only turn them on when needed
    - Check against the list of unique network devices connected to each switch
    - Create the configuration
      - Interfaces to be changed
      - The VLAN the interfaces needs to be changed to
      - Any special configuration necessary
        - Host-mode multi-host for unmanaged switches or hubs
        - Trunk interface for honeypots
        - Voice VLAN configured on the access VLAN for voice gateways and panic buttons
        - Spanning-tree bpduguard disabled for vendor managed switches
        - Add descriptions for these special configurations


  - IDF cut-over
    - Schedule dates and times with
      - Hardware Team
      - Offices and sections
      - Management
    - Submit change notification(s) for approval
    - Call campus security and remind them of the outage
	  - Security cameras will go offline
	  - Badge readers will stop working
	  - On some doors, the mag locks will prevent keys from being used
    - Remove old IDF switches if IDF cabinet is full
      - Rack new IDF switches
    - Move fiber uplinks from the old MDF switch to the new MDF switch
    - Move fiber uplinks from old IDF switch to new IDF switch
    - Move copper cables from old IDF switch to new IDF switch
    - Power up
    - Unrack the old IDF switches
    - On the new MDF switches
      - Verify
        - Etherchannel
          - sh etherc sum
        - Trunk interfaces
          - sh int trunk
    - On each IDF switch
      - Verify
        - Remote access
        - Etherchannel
          - sh etherc sum
        - VLAN information updated via VTP
          - Sometimes, this will not happen
          - On the old MDF switch that is the VTP primary, run the global exec command "vtp primary"
          - sh vlan bri
          - sh vtp status
        - Trunk interfaces
          - sh int trunk
        - NTP
          - Usually takes 5 - 10 minutes to synchronize
          - sh ntp ass
          - sh ntp ass detail
          - sh ntp status
        - IP reachability
          - Ping campus DNS server
            - ping 8.8.8.8
          - Ping colo DNS server
            - ping 8.8.4.4
        - Check inventory
          - Did something get damaged or stop working?
            - sh inventory
        - Check power supply and fans
          - Did something get damaged or stop working?
            - sh environment all
        - Apply the IDF after cut-over configuration changes
          - For the non-IDF VLAN interfaces
          - DON'T FORGET THIS STEP
          - YOU WILL BE CALLED AT 5AM THE NEXT MORNING IF YOU FORGET
        - Check interfaces
          - Any err-disabled?
          - Check repeatedly for at least 10 minutes
            - sh int status
        - Check dot1x
          - Any auth failed?
          - Check repeatedly for at least 10 minutes
            - sh access-session
        - Check PoE
          - Any PoE failures or did the switch run out of available watts?
            - sh power inline
        - Check DHCP
          - Anything receiving DHCP IP addresses?
            - sh ip dhcp snooping binding
        - Check logging
          - Any dot1x failures?
            - Are there any interface need to be configured for host-mode multi-host?
          - Are any interfaces flapping?
          - Are there any unusual logging messages?
          - Watch the logging messages for at least 10 minutes after the cut-over
            - term mon
        - Check UPS reachability
    - Network monitoring
      - Delete the old network monitoring network object
        - You can do some low lever SQL commands to change the device type and a few other 
	  things that do not get updated on their own
        - It's easier to delete the old object
      - Add the new network monitoring network object
        - Configure the resources monitored
        - Configure the custom properties
        - Configure NCM
    - Walk around and verify IP phones are working
      - Call your cell phone
      - Call the IP phone from your cell phone
    - Verify building badge readers work
      - These will not work if you forgot to apply the IDF after cut-over configuration changes
    - Verify wireless works
	- Call campus security and let them know the outage is complete


  - Create MDF cut-over configuration changes
    - Cutting over routing and switching from the old MDFs to the new MDFs
    - Change VLAN 1 IP address
      - If not using HSRP
        - change to .1
      - If using HSRP
        - change to .2 and .3
    - SVIs
      - no shut
    - Spanning-tree
      - Change the spanning-tree VLAN priority to become the root bridge


  - MDF switch and routing cut-over
    - Schedule dates and times with
      - Hardware Team
      - Offices and sections
      - Management
    - Submit change notification(s) for approval
    - Call campus security and remind them of the outage
	  - Security cameras may go offline
	  - Badge readers may stop working
	  - On some doors, the mag locks may prevent keys from being used
	- Endpoints typically lose network connectivity for a few seconds as routing and 
	  switch converges
    - What we've been doing
      - Turn off the old MDFs
    - Apply the MDF cut-over configuration changes
    - Change ISE network object if necessary
      - It should use the old ISE network object
	  - Do this ahead of time
    - VTP
      - Run "vtp primary"
      - Make sure VLAN information is correct
      - Reapply the vlan configuration if necessary
        - sh vlan bri
      - Delete any VLANs and SVIs that are no longer necessary
    - Verify
      - Remote access
      - NTP
      - Routing neighbor state
      - Routing table
      - HSRP
        - If configured
          - sh standby bri
      - Spanning-tree
        - sh spanning-tree | in ^VLAN|root
      - Trunk links
        - sh int trunk
      - IP reachability
        - Ping campus DNS server
          - ping 8.8.8.8
        - Ping colo DNS server
          - ping 8.8.4.4
      - Logging messages
      - Run the ping all tcl script from a network core device
        - Includes a list of all VLANs / SVIs
        - Includes a list of uplink IP address
        - Incluses a list of all loopbacks
        - Includes a list of all IDF management IP addresses
        - Includes a list of IP addresses of any other critical devices in the building
      - Run the ping all cmd script from a bounce server
        - Includes a list of all VLANs / SVIs
        - Includes a list of uplink IP address
        - Incluses a list of all loopbacks
        - Includes a list of all IDF management IP addresses
        - Includes a list of IP addresses of any other critical devices in the building
      - Check the honeypot
        - See if it's up
          - namp -n -sn -PE 10.x.x.0/24
          - nmap -A -T4 10.x.x.x 
            - IP address of the honeypot
        - IT security will ask about it the next morning
    - Network monitoring
      - Delete the old network monitoring network object
      - Add the new network monitoring network object
        - Configure the resources monitored
        - Configure the custom properties
        - Configure NCM
    - Check network monitoring
      - Make sure all devices in the building are up
    - Remove temporary old IDF to new IDF optics and cabling


  - Campus distribution switch cleanup
    - Default interface configuration used for the old MDF
    - Update routing configuration
      - Remove old neighbor configuration
      - Remove old uplinks from being advertised
    - Move the uplinks from the temporary interfaces to the interfaces used by the old MDF switches
      - Copy existing configuration
      - Default the interface being moved
      - Apply the configuration to the interface the link is being moved to
      - Move the optics from the old interface to the new interface
      - Verify
        - Link up / up
        - IP connectivity on the link
        - Routing neighbor is up
        - Routing information is being exchanged


  - MDF routing cleanup
    - Delete the temporary MED route-map configuration


  - Core network cleanup
    - Update routing configuration
      - Remove old uplinks from redistribution
      - Remove old loopbacks from redistribution


  - Temporary configurations removed
    - MDF
      - Re-IP VLAN 1 to permanent IP address
        - This really should have been done during the MDF routing and switching cut-over
        - If not
  	      - Update network monitoring
          - Delete the temporary network device object from ISE
          - Update the SecureCRT session configuration as necessary
      - Change source-interface VLAN 1 for services
        - RADIUS
        - TACACS
        - logging
        - NTP
        - SSH
        - HTTP server for ISE
        - HTTPS server for ISE
      - Default management interface
      - Shutdown management interface
      - Remove the temporary management to IDFMDF cable
    - IDF
      - Re-IP any temporary IP addresses
        - update network monitoring
        - Delete the temporary network device objects from ISE
        - Update the ISE network device objects if necessary
          - Switch model
        - Update your SecureCRT session configuration as necessary
      - Verify
        - Remote access
        - NTP
          - Usually takes 5 - 10 minutes to synchronize
          - sh ntp ass
          - sh ntp ass detail
          - sh ntp status
        - IP reachability
          - ping campus DNS server
            - ping 8.8.8.8
          - Ping colo DNS server
            - ping 8.8.4.4
        - Notify the Network Team when complete
        - Notify the Telecom Team when complete
          - So they can update the CER information	


  - Wipe old IDF switches
    - Delete configuration
	- Delete vlan.dat
	- Delete any copies of the configuration created by the archive configuration or by 
	  other network engineers
	- Delete any core dumps files
	- Delete any crash files
	- Delete any copies of show tech


  - Unrack MDF switches
    - Usually wait a week or two, just in case
    - Wipe the MDF switches first
    - Sometimes, it's easier to wipe the MDFs while they are still racked
      - Especially if they use C19 / C20 power cables
    - Go ahead, use Google to figure out how to delete the vlan.dat on that old Catalyst 4500 
      or 6500 dinosaur that was on the network for 20 years
    - Ask management if you can have it to turn into a beer fridge


  - Unrack old UPS
    - If replaced


  - Check PoE usage
    - On all IDFs to see if a secondary power supply is necessary
      - You added spares to the BOM, right?


  - DHCP clean up
    - Rename IP address pools if any of the IDFs were renumbered
    - Delete any IP address pools that are not longer used after the tech refresh


  - IP Address Management clean up
    - Delete the old IDF uplink subnets
    - Delete the old MDF loopbacks
    - Delete any VLANs that are no longer being used


  - Documentation update and clean up
    - Doku
      - Switch list
      - VLAN list
      - Campus distribution information
      - BGP / routing information
      - UPS list
    - Review
      - Network diagram is up to date
      - IDF layout is up to date
    - Take pictures of the MDF
      - For documentation
      - For Twitter


  - Miscellaneous
    - If you grabbed any folding chairs to use in the MDFs, put them back
    - Return any boom lifts or scissor lifts after use, don't forget to plug them in
    - Return any safety harness if borrowed
```
