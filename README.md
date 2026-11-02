# Software-Project-Case-Study
"""knight_demo.py - simplified model of the Knight Capital failure.

Illustrative only: this is NOT the real SMARS / Power Peg code."""

# ---------- Fault 1: partial deployment + re-used flag ----------
class Server:
    def __init__(self, name, has_new_code):
        self.name = name
        self.has_new_code = has_new_code      # was the new release copied here?

    def handle_flag(self, flag_on):
        # The same flag now means "RLP" in new code but "Power Peg" in old code
        if not flag_on:
            return "normal trading"
        if self.has_new_code:
            return "RLP (new feature)"
        return "POWER PEG (old test code!)"

# Technician deploys to 8 servers but misses server 8 (manual process)
servers = [Server(f"server-{i}", has_new_code=(i != 8)) for i in range(1, 9)]

print("--- Fault 1: partial deployment ---")
for s in servers:
    print(f"{s.name}: flag ON -> {s.handle_flag(True)}")

# ---------- Fault 2: Power Peg never stops sending orders ----------
def power_peg(parent_qty, counter_works, kill_switch_at=None):
    """Sells child orders until the parent order is filled."""
    child_size, bid, ask = 100, 10.00, 10.05   # buys at ask, sells at bid
    cumulative, children, loss = 0, 0, 0.0
    while cumulative < parent_qty:
        children += 1
        loss += child_size * (ask - bid)       # buy high, sell low
        if counter_works:
            cumulative += child_size           # old, correct behaviour
        # BUG: with counter_works=False the fill count never grows
        if kill_switch_at and children >= kill_switch_at:
            break                              # humans finally pull the plug
    return children, loss

print("\n--- Fault 2: runaway order loop ---")
n, loss = power_peg(1000, counter_works=True)
print(f"Counter works : {n} child orders, loss = ${loss:,.2f}")
n, loss = power_peg(1000, counter_works=False, kill_switch_at=1_000_000)
print(f"Counter broken: {n:,} child orders, loss = ${loss:,.2f}")

# ---------- What a safe deployment should do ----------
def safe_to_enable_flag(servers):
    return all(s.has_new_code for s in servers)

print("\n--- Safe deployment check ---")
print("Enable flag?", "YES" if safe_to_enable_flag(servers) else "NO - server missing new code")
