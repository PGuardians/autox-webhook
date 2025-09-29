import re
import html
import json
from textwrap import dedent
from RestrictedPython import compile_restricted_exec
from RestrictedPython.Eval import default_guarded_getitem, default_guarded_getiter, default_guarded_getattr

def get_next_node(execution, id):
    for node in execution['nodes']:
        if node['id'] == id:
            return  node

def parse_crm_style_dict(raw_str):
    try:
        # Remove enclosing braces
        raw_str = raw_str.strip()[1:-1]

        # Replace <mailto:...|...> or <tel:...|...> with just the visible value
        raw_str = re.sub(r'<[^:]+:([^|]+)\|([^>]+)>', r'\2', raw_str)

        # Handle keys and values safely
        result = {}
        key_value_pairs = []
        current = ""
        in_value = False

        for part in raw_str.split(","):
            if ":" in part and not in_value:
                key_value_pairs.append(current)
                current = part
            else:
                current += "," + part
        key_value_pairs.append(current)

        for pair in key_value_pairs:
            if ":" not in pair:
                continue
            key, value = pair.split(":", 1)
            result[key.strip()] = value.strip()
        print("RESULT", result, "RESULT")
        return result
    except Exception as e:
        print(e)
        return None


def clean_quoted_dict(raw_dict):
    cleaned = {}
    for key, value in raw_dict.items():
        # Remove surrounding quotes from key and value
        key = key.strip('"\'')
        value = value.strip('"\'')
        
        # Unescape HTML entities like &amp;
        value = html.unescape(value)
        
        # Optional: remove wrapping <...> from URLs/emails/etc.
        if value.startswith("<") and value.endswith(">"):
            value = value[1:-1]

        cleaned[key] = value

    return cleaned

def compileCode(code, input,true_branch, false_branch):
    try:
        print('[INFO] Compiling code', code, input)
        code = dedent(code)
        compiled_code = compile_restricted_exec(code)

        safe_globals = {
            "__builtins__": {},
            "_getiter_": default_guarded_getiter,
            "_getitem_": default_guarded_getitem,
            "_getattr_": default_guarded_getattr,
        }

        safe_locals = {}
        input_data = input
        if isinstance(input_data, str):
            try:
                input_data = json.loads(input_data)
            except json.JSONDecodeError:
                raise ValueError("Invalid JSON string in 'input' field.")
            
        if compiled_code and compiled_code.code:
            exec(compiled_code.code, safe_globals, safe_locals)
            result = safe_locals["run"](input_data)
            next_node_id = true_branch if result else false_branch
            return {"result": result, "nextNodeId": next_node_id}
        else:
            if compiled_code.errors:
                print(compiled_code.errors)
                raise SyntaxError(compiled_code.errors)
            else:
                raise Exception("Something went wrong")

    except Exception as e:
        print("Execution Error:", str(e))
