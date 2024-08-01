import os
import asyncio
from dotenv import load_dotenv
import logging
from web3 import Web3
import signal
import discord
from discord.ext import tasks

load_dotenv()

logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

INFURA_URL = os.getenv('INFURA_URL')
CONTRACT_ADDRESS = os.getenv('CONTRACT_ADDRESS')
DISCORD_TOKEN = os.getenv('DISCORD_TOKEN')
DISCORD_CHANNEL_ID_1 = int(os.getenv('DISCORD_CHANNEL_ID_1'))

contract_abi = [
    {"inputs":[{"internalType":"address","name":"_agiTokenAddress","type":"address"},{"internalType":"string","name":"_baseIpfsUrl","type":"string"},{"internalType":"address","name":"_ensAddress","type":"address"},{"internalType":"address","name":"_nameWrapperAddress","type":"address"},{"internalType":"bytes32","name":"_clubRootNode","type":"bytes32"},{"internalType":"bytes32","name":"_agentRootNode","type":"bytes32"},{"internalType":"bytes32","name":"_validatorMerkleRoot","type":"bytes32"},{"internalType":"bytes32","name":"_agentMerkleRoot","type":"bytes32"}],"stateMutability":"nonpayable","type":"constructor"},
    {"anonymous":False,"inputs":[{"indexed":True,"internalType":"address","name":"nftAddress","type":"address"},{"indexed":False,"internalType":"uint256","name":"payoutPercentage","type":"uint256"}],"name":"AGITypeUpdated","type":"event"},
    {"anonymous":False,"inputs":[{"indexed":True,"internalType":"address","name":"owner","type":"address"},{"indexed":True,"internalType":"address","name":"approved","type":"address"},{"indexed":True,"internalType":"uint256","name":"tokenId","type":"uint256"}],"name":"Approval","type":"event"},
    {"anonymous":False,"inputs":[{"indexed":True,"internalType":"address","name":"owner","type":"address"},{"indexed":True,"internalType":"address","name":"operator","type":"address"},{"indexed":False,"internalType":"bool","name":"approved","type":"bool"}],"name":"ApprovalForAll","type":"event"},
    {"anonymous":False,"inputs":[{"indexed":False,"internalType":"uint256","name":"_fromTokenId","type":"uint256"},{"indexed":False,"internalType":"uint256","name":"_toTokenId","type":"uint256"}],"name":"BatchMetadataUpdate","type":"event"},
    {"anonymous":False,"inputs":[{"indexed":False,"internalType":"uint256","name":"jobId","type":"uint256"},{"indexed":False,"internalType":"address","name":"resolver","type":"address"},{"indexed":False,"internalType":"string","name":"resolution","type":"string"}],"name":"DisputeResolved","type":"event"},
    {"anonymous":False,"inputs":[{"indexed":False,"internalType":"uint256","name":"jobId","type":"uint256"},{"indexed":False,"internalType":"address","name":"agent","type":"address"}],"name":"JobApplied","type":"event"},
    {"anonymous":False,"inputs":[{"indexed":False,"internalType":"uint256","name":"jobId","type":"uint256"}],"name":"JobCancelled","type":"event"},
    {"anonymous":False,"inputs":[{"indexed":False,"internalType":"uint256","name":"jobId","type":"uint256"},{"indexed":False,"internalType":"address","name":"agent","type":"address"},{"indexed":False,"internalType":"uint256","name":"reputationPoints","type":"uint256"}],"name":"JobCompleted","type":"event"},
    {"anonymous":False,"inputs":[{"indexed":False,"internalType":"uint256","name":"jobId","type":"uint256"},{"indexed":False,"internalType":"address","name":"agent","type":"address"}],"name":"JobCompletionRequested","type":"event"},
    {"anonymous":False,"inputs":[{"indexed":False,"internalType":"uint256","name":"jobId","type":"uint256"},{"indexed":False,"internalType":"string","name":"ipfsHash","type":"string"},{"indexed":False,"internalType":"uint256","name":"payout","type":"uint256"},{"indexed":False,"internalType":"uint256","name":"duration","type":"uint256"},{"indexed":False,"internalType":"string","name":"details","type":"string"}],"name":"JobCreated","type":"event"},
    {"anonymous":False,"inputs":[{"indexed":False,"internalType":"uint256","name":"jobId","type":"uint256"},{"indexed":False,"internalType":"address","name":"validator","type":"address"}],"name":"JobDisapproved","type":"event"},
    {"anonymous":False,"inputs":[{"indexed":False,"internalType":"uint256","name":"jobId","type":"uint256"},{"indexed":False,"internalType":"address","name":"disputant","type":"address"}],"name":"JobDisputed","type":"event"},
    {"anonymous":False,"inputs":[{"indexed":False,"internalType":"uint256","name":"jobId","type":"uint256"},{"indexed":False,"internalType":"address","name":"validator","type":"address"}],"name":"JobValidated","type":"event"},
    {"anonymous":False,"inputs":[{"indexed":True,"internalType":"bytes32","name":"newMerkleRoot","type":"bytes32"}],"name":"MerkleRootUpdated","type":"event"},
    {"anonymous":False,"inputs":[{"indexed":False,"internalType":"uint256","name":"_tokenId","type":"uint256"}],"name":"MetadataUpdate","type":"event"},
    {"anonymous":False,"inputs":[{"indexed":True,"internalType":"uint256","name":"tokenId","type":"uint256"}],"name":"NFTDelisted","type":"event"},
    {"anonymous":False,"inputs":[{"indexed":True,"internalType":"uint256","name":"tokenId","type":"uint256"},{"indexed":True,"internalType":"address","name":"employer","type":"address"},{"indexed":False,"internalType":"string","name":"tokenURI","type":"string"}],"name":"NFTIssued","type":"event"},
    {"anonymous":False,"inputs":[{"indexed":True,"internalType":"uint256","name":"tokenId","type":"uint256"},{"indexed":True,"internalType":"address","name":"seller","type":"address"},{"indexed":False,"internalType":"uint256","name":"price","type":"uint256"}],"name":"NFTListed","type":"event"},
    {"anonymous":False,"inputs":[{"indexed":True,"internalType":"uint256","name":"tokenId","type":"uint256"},{"indexed":True,"internalType":"address","name":"buyer","type":"address"},{"indexed":False,"internalType":"uint256","name":"price","type":"uint256"}],"name":"NFTPurchased","type":"event"},
    {"anonymous":False,"inputs":[{"indexed":True,"internalType":"address","name":"previousOwner","type":"address"},{"indexed":True,"internalType":"address","name":"newOwner","type":"address"}],"name":"OwnershipTransferred","type":"event"},
    {"anonymous":False,"inputs":[{"indexed":False,"internalType":"address","name":"claimant","type":"address"},{"indexed":False,"internalType":"string","name":"subdomain","type":"string"}],"name":"OwnershipVerified","type":"event"},
    {"anonymous":False,"inputs":[{"indexed":False,"internalType":"address","name":"account","type":"address"}],"name":"Paused","type":"event"},
    {"anonymous":False,"inputs":[{"indexed":False,"internalType":"string","name":"reason","type":"string"}],"name":"RecoveryInitiated","type":"event"},
    {"anonymous":False,"inputs":[{"indexed":False,"internalType":"address","name":"user","type":"address"},{"indexed":False,"internalType":"uint256","name":"newReputation","type":"uint256"}],"name":"ReputationUpdated","type":"event"},
    {"anonymous":False,"inputs":[{"indexed":True,"internalType":"address","name":"contributor","type":"address"},{"indexed":False,"internalType":"uint256","name":"amount","type":"uint256"}],"name":"RewardPoolContribution","type":"event"},
    {"anonymous":False,"inputs":[{"indexed":True,"internalType":"bytes32","name":"newRootNode","type":"bytes32"}],"name":"RootNodeUpdated","type":"event"},
    {"anonymous":False,"inputs":[{"indexed":True,"internalType":"address","name":"from","type":"address"},{"indexed":True,"internalType":"address","name":"to"},{"indexed":True,"internalType":"uint256","name":"tokenId","type":"uint256"}],"name":"Transfer","type":"event"},
    {"anonymous":False,"inputs":[{"indexed":False,"internalType":"address","name":"account","type":"address"}],"name":"Unpaused","type":"event"}
]

web3 = Web3(Web3.HTTPProvider(INFURA_URL))
contract_address = Web3.to_checksum_address(CONTRACT_ADDRESS)
contract = web3.eth.contract(address=contract_address, abi=contract_abi)

intents = discord.Intents.default()
client = discord.Client(intents=intents)

@tasks.loop(seconds=300)
async def check_for_events():
    print("Checking for new events...")
    latest_block = web3.eth.block_number
    event_filter = contract.events.JobCreated.create_filter(fromBlock=latest_block - 10)
    events = event_filter.get_all_entries()
    
    channel = client.get_channel(DISCORD_CHANNEL_ID_1)
    for event in events:
        event_details = event['args']
        ipfs_hash = event_details['ipfsHash']
        ipfs_link = f"https://ipfs.io/ipfs/{ipfs_hash}"
        message = (
            "@772131161215336489 @892954846934761593\n"
            "New job created\n"
            f"jobId: {event_details['jobId']}\n"
            f"IPFS Link: {ipfs_link}\n"
            f"payout: {event_details['payout'] / 1e18}\n"
            f"duration: {event_details['duration']}\n"
            f"details: {event_details['details']}"
        )
        await channel.send(message)

@client.event
async def on_ready():
    print(f'{client.user} has connected to Discord!')
    channel = client.get_channel(DISCORD_CHANNEL_ID_1)
    await channel.send(f"@emperor_agi  Bot is now online and monitoring for new events!")
    check_for_events.start()

@client.event
async def on_message(message):
    if message.content == "!stopjob":
        await message.channel.send("Stopping the bot...")
        await shutdown()

async def shutdown():
    print("Received exit signal, shutting down...")
    check_for_events.cancel()
    tasks = [t for t in asyncio.all_tasks() if t is not asyncio.current_task()]
    [task.cancel() for task in tasks]
    print(f"Cancelling {len(tasks)} outstanding tasks")
    await asyncio.gather(*tasks, return_exceptions=True)
    channel = client.get_channel(DISCORD_CHANNEL_ID_1)
    try:
        await channel.send("@772131161215336489  Bot is shutting down.")
    except Exception as e:
        logging.error(f"Failed to send shutdown message: {e}")
    print("Closing Discord connection...")
    await client.close()
    print("Shutdown complete.")

def signal_handler(sig, frame):
    loop = asyncio.get_event_loop()
    loop.create_task(shutdown())
    loop.stop()

def main():
    print("Starting Discord bot...")
    loop = asyncio.get_event_loop()
    signal.signal(signal.SIGINT, signal_handler)
    signal.signal(signal.SIGTERM, signal_handler)
    try:
        loop.run_until_complete(client.start(DISCORD_TOKEN))
    except KeyboardInterrupt:
        print("Keyboard interrupt received.")
    finally:
        loop.run_until_complete(shutdown())
        loop.close()
        print("Bot has been shut down.")

if __name__ == "__main__":
    main()
