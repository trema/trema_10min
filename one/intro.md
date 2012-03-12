<!SLIDE title-slide>
# Trema in 10 Minutes ##########################################################

### Yasuhito Takamiya  `@yasuhito`


<!SLIDE small incremental transition=uncover>
# What's Trema? ################################################################

* New OpenFlow programming framework in Ruby and C
  * GPL2
  * <http://github.com/trema/trema>
* Designed to be highly productive for this "Post-Rails" era
  * <i>Run It Quick</i>: Tight loop of coding, run, and debug
  * <i>Convention over Coding</i>: Write it short
  * <i>Integrated Unit-Testing</i>
* Let's go through the entire cycle of development using Trema


<!SLIDE small>
# Trema Controller - A single line of code ####################################

	@@@ ruby
	def ExampleController < Controller; end

## Simple and complete but not so much useful or interesting

<br /><br />

## Incremental development - Add a packet_in handler

	@@@ ruby
	def ExampleController < Controller
	  def packet_in ...
	end


<!SLIDE small>
# One Basic Command: `trema run` ###############################################

	$ trema run [controller-file]

* Starts the controller process/application in foreground
* Test your controller right away without compilation
* Enables the tight loop of "Coding, test, and debug"


<!SLIDE smaller>
# Run on what/where?<br />Network DSL (Domain Specific Language) ###################

	$ trema run learning-switch.rb -c network.conf

* Runs your controller on a virtualized network described in the .conf file
* You can specify and construct any arbitrary virtual topology
* You can develop with your laptop, no need for physical switches!
* The controller developed in virtualized network can be seamlessly deployed into real network


<!SLIDE smaller>
# Network DSL Example (network.conf) ###########################################

	@@@ ruby
	# Add one virtual switch
	vswitch { dpid "0xabc" }
	# Add two virtual hosts
	vhost "host1"
	vhost "host2"
	# Then connect them to the switch 0xabc
	link "0xabc", "host1"
	link "0xabc", "host2"


<!SLIDE smaller>
# Debugging on Virtual Network #################################################

	$ trema send_packets --source host1 --dest host2
	$ trema show_stats host1
	$ trema show_stats host2
	$ trema send_packets --source host2 --dest host1
	$ trema dump_flows 0xabc


<!SLIDE small>
# Convention over Coding #######################################################

	@@@ ruby
	class MyController < Controller
	  def start  # start-up event handler
	    # ...
	  end
	      
	  def packet_in dpid, msg  # Packet-in received handler
	    # ...
	  end
	
	  # ...
	end

* Coding conventions for concise and compact code
  * All controllers are defined as a class derived from `Controller`
  * "handler name == message name" just like Rails controller


<!SLIDE smaller>
# Flow-Mod API #################################################################

	@@@ ruby
	class LearningSwitch < Controller
	
	  # ...
	
	  def packet_in dpid, message
	    send_flow_mod_add(
	      dpid,
	      :match => ExactMatch.from( message ),
	      :actions => ActionOutput.new( message.port_no + 1 )
	    )
	    # ...
	  end
	
	  # ...
	
	end


<!SLIDE smaller>
# Syntactic Sugar: `ExactMatch.from()` #########################################

	@@@ ruby
	ExactMatch.from( message )

# vs.

	@@@ ruby
	Match.new(
	  :in_port => message.in_port,
	  :nw_src => message.nw_src,
	  :nw_dst => message.nw_dst,
	  :tp_src => message.tp_src,
	  :tp_dst => message.tp_dst,
	  :dl_src => message.dl_src,
	  :dl_dst => message.dl_dst,
	  ...
	)


<!SLIDE smaller>
# FlowMod API: Trema vs. NOX Python #########################################################

	@@@ ruby
	# Trema
	send_flow_mod_add(
	  dpid,
	  :match => ExactMatch.from( message ),
	  :actions => ActionOutput.new( port_no )
	)

# vs.

	@@@ python
	# NOX Python
	inst.install_datapath_flow(
	  dpid,
	  extract_flow(packet),
	  CACHE_TIMEOUT, 
	  openflow.OFP_FLOW_PERMANENT,
	  [[openflow.OFPAT_OUTPUT, [0, prt[0]]]],
	  bufid,
	  openflow.OFP_DEFAULT_PRIORITY,
	  inport,
	  buf
	)


<!SLIDE smaller>
# RSpec Integration ############################################################

* Trema offers a unit-testing framework integrated with RSpec
* Assertions and expectations over controllers, switches and hosts
* You can write tests using RSpec-style fluent API
  * `vswitch( "0xabc" ).should have( 1 ).flows`
  * `controller( "RepeaterHub" ).should_receive( :packet_in )`
  * `vhost( "host1" ).rx_stats.n_pkts.should == 2`


<!SLIDE small incremental transition=uncover>
# Trema: "OpenFlow like Rails" #################################################

* <i>Run It Quick</i>: Tight loop of coding, run, and debug
  * Virtual network DSL
  * `trema {run, send_packets, show_stats, dump_flows}`
* <i>Convention Over Coding</i>: Write it short
  * Auto handler dispatch by naming convention
  * Syntactic sugars: `ExactMatch.from`
  * Default options: `send_flow_mod_add`
* <i>RSpec Integration</i>
  * Write unit-tests of network components


<!SLIDE small>
# Sources ######################################################################

* Trema: <http://github.com/trema/>
* Web Page: <http://trema.github.com/trema/>
* Twitter: <http://twitter.com/trema_news>
* Mailing List: <https://groups.google.com/group/trema-dev>
* Bugs: <https://github.com/trema/trema/issues>
